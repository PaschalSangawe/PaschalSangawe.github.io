---
title: "PortSwigger SQL Injection Labs — Part 5: WAF Bypass via XML Encoding"
date: 2026-09-22 06:40:00 +0000
categories: [Web Penetration Testing]
tags: [SQL Injection, PortSwigger, Web Security Academy, WAF Bypass]
description: "Bypassing a SQL injection filter by obfuscating a UNION SELECT payload with XML character entities, using Burp's Hackvertor extension."
author: Paschal Sangawe
toc: true
---

The final lab (17) is different in two ways: the injection point is inside an **XML body**, and a **WAF** inspects the request for SQL keywords. The trick is to encode the payload as XML character entities, which the XML parser decodes *after* the WAF has already inspected the raw request.

## Lab 17 — SQL injection with filter bypass via XML encoding
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<?xml version="1.0" encoding="UTF-8"?>`.
> _Save as_ `images/portswigger-sqli-part-5-filter-bypass-xml-encoding/1.png` _then replace this block with_ `![Lab 17 — SQL injection with filter bypass via XML encoding](/images/portswigger-sqli-part-5-filter-bypass-xml-encoding/1.png)`_._


**Difficulty:** Practitioner
**Goal:** Extract the administrator credentials through a stock-check request and log in.

### Step 1 — Identify the injection point

The stock-check feature sends the `productId` and `storeId` to `POST /product/stock` as XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck>
  <productId>1</productId>
  <storeId>1</storeId>
</stockCheck>
```

Send this request to Burp Repeater and confirm the `storeId` is evaluated as a number by using an arithmetic expression:

```xml
<storeId>1+1</storeId>
```

If the response returns the stock for a different store, the value is being evaluated server-side — a strong hint that it reaches the SQL query.

### Step 2 — Hit the WAF

Try a straightforward `UNION` to discover the column count:

```xml
<storeId>1 UNION SELECT NULL</storeId>
```

The request is now blocked as a suspected attack. That confirms a signature-based filter is looking for SQL keywords in the request body.

### Step 3 — Bypass with XML entities

Because the body is XML, we can represent characters as entities (`&#x53;` for `S`, etc.). The WAF sees only entity references, while the XML parser reconstructs the original string before it reaches the SQL layer.

With the **Hackvertor** extension installed (BApp Store):

1. Select the value of `<storeId>` inside the XML in Repeater.
2. Right-click → **Extensions → Hackvertor → Encode → hex_entities** (or `dec_entities`).

Hackvertor wraps the value like this:

```xml
<storeId><@hex_entities>1 UNION SELECT NULL</@hex_entities></storeId>
```

Resend. The response is now normal, which shows the WAF was bypassed.

### Step 4 — Build the exploit

The query only returns a single column (adding a second column makes the application report `0 units`, i.e. an error). So concatenate username and password into one column. Oracle/PostgreSQL/MSSQL use `||` (if the lab backend is MySQL, use `CONCAT`):

```xml
<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users</@hex_entities></storeId>
```

Send it and the response contains the usernames and passwords separated by `~`.

> **Picture goes here (#2).** Repeater request with the `hex_entities` wrapper and the response containing `administrator~<password>`.
> _Save as_ `images/portswigger-sqli-part-5-filter-bypass-xml-encoding/2.png` _then replace this block with_ `![Step 4 — Build the exploit](/images/portswigger-sqli-part-5-filter-bypass-xml-encoding/2.png)`_._

### Step 5 — Log in

Use the recovered administrator password to log in and solve the lab.

## Why this works

- **Context matters:** any input that is parsed into a SQL query is fair game, including JSON and XML bodies — not just query strings.
- **Encoding defeats weak filters:** a filter that matches literal SQL keywords misses the same keywords when they are XML entity references, because decoding happens later in the pipeline.
- **Defence is not parameterization:** WAFs and blocklists reduce noise but never fix the root cause. The application still concatenates the value into the query.

## Series wrap-up

Across the 17 labs the exploitation funnel is consistent:

1. **Prove the injection** with a quote, `OR 1=1`, or a comment (`-- ` / `#`).
2. **Characterise the database** (Oracle vs MySQL/MSSQL/PostgreSQL) and the query shape (column count, text columns).
3. **Pick a channel:** in-band `UNION SELECT` when data is reflected; boolean, error, timing or OAST when it is blind.
4. **Automate extraction** with Burp Intruder once a reliable per-character oracle exists.
5. **Escalate** to credential theft and account takeover.

And the fix is always the same: **parameterised queries / prepared statements**, with an allow-list for the parts of a query (table names, `ORDER BY`) that cannot be parameterised.

Thanks for reading. If you spot a better payload or a faster extraction method, reach out on [X](https://twitter.com/KidayoLade60178) or [LinkedIn](https://www.linkedin.com/in/paschal-sangawe-a386592a4/).

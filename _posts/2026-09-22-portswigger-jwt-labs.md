---
title: "PortSwigger JWT Labs — Complete Walkthrough"
date: 2026-09-22 08:10:00 +0000
categories: [Web Penetration Testing]
tags: [JWT, PortSwigger, Web Security Academy, Authentication]
description: "All 8 PortSwigger JWT labs: unverified signatures, alg:none, weak keys, jwk/jku injection, kid path traversal and RS256-to-HS256 algorithm confusion."
author: Paschal Sangawe
toc: true
---

JSON Web Tokens (JWTs) are widely used for stateless sessions and API authorization. Because the token itself carries the claims the server trusts, any weakness in how the signature is created or verified turns directly into authentication bypass. This is one of the most reliable ways to escalate to admin in real applications.

This post covers all **8 PortSwigger JWT labs**. Most solutions use the **JWT Editor** Burp extension (BApp Store), which lets you decode, edit, sign and attack tokens in Repeater.

## Background — JWT anatomy

A JWT has three Base64URL parts separated by dots:

```text
<header>.<payload>.<signature>
```

```json
// header
{ "alg": "HS256", "typ": "JWT" }

// payload
{ "sub": "wiener", "exp": 1700000000 }
```

The server is supposed to verify the signature over `header.payload`. Every lab here breaks that assumption in a different way.

## Lab 1 — JWT authentication bypass via unverified signature
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `/admin`.
> _Save as_ `images/portswigger-jwt-labs/1.png` _then replace this block with_ `![Lab 1 — JWT authentication bypass via unverified signature](/images/portswigger-jwt-labs/1.png)`_._


**Difficulty:** Apprentice
**Goal:** Reach `/admin` by editing a claim without re-signing.

The server decodes the token but never verifies the signature.

1. Log in and send `GET /my-account` to Repeater. The session cookie is a JWT whose `sub` is `wiener`.
2. Change the path to `/admin` → denied.
3. In the JWT Editor's payload view, change `sub` to `administrator` and **Apply changes** (do not re-sign).
4. Send `/admin` → access granted.
5. Request `/admin/delete?username=carlos`.

> **Picture goes here (#2).** The edited `sub: administrator` token accessing the admin panel.
> _Save as_ `images/portswigger-jwt-labs/2.png` _then replace this block with_ `![Lab 1 — JWT authentication bypass via unverified signature](/images/portswigger-jwt-labs/2.png)`_._

## Lab 2 — JWT authentication bypass via flawed signature verification
> **Picture goes here (#3).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `alg: none`.
> _Save as_ `images/portswigger-jwt-labs/3.png` _then replace this block with_ `![Lab 2 — JWT authentication bypass via flawed signature verification](/images/portswigger-jwt-labs/3.png)`_._


**Difficulty:** Apprentice
**Goal:** Exploit an `alg: none` acceptance.

Some libraries treat `none` as "no signature required".

1. Change `sub` to `administrator`.
2. Change the header `alg` to `none`.
3. **Remove the signature** but keep the trailing dot: `header.payload.`
4. Send `/admin` → access granted. Delete `carlos`.

> **Note:** some parsers want `none`, `None`, or `NONE` — try variants if blocked.

> **Picture goes here (#4).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-jwt-labs/4.png` _then replace this block with_ `![Lab 2 — JWT authentication bypass via flawed signature verification](/images/portswigger-jwt-labs/4.png)`_._

## Lab 3 — JWT authentication bypass via weak signing key
> **Picture goes here (#5).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list`.
> _Save as_ `images/portswigger-jwt-labs/5.png` _then replace this block with_ `![Lab 3 — JWT authentication bypass via weak signing key](/images/portswigger-jwt-labs/5.png)`_._


**Difficulty:** Practitioner
**Goal:** Crack a weak HMAC secret and re-sign an admin token.

The token is signed with HS256 using a guessable secret.

1. Copy the JWT and brute-force the secret with hashcat:

```bash
hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list
```

> **Picture goes here (#6).** Capture this request/response or command step in Burp/terminal. Key line: `hashcat -a 0 -m 16500 <YOUR-JWT> /path/to/jwt.secrets.list`.
> _Save as_ `images/portswigger-jwt-labs/6.png` _then replace this block with_ `![Lab 3 — JWT authentication bypass via weak signing key](/images/portswigger-jwt-labs/6.png)`_._


The secret is `secret1`.
2. Base64-encode the secret. In **JWT Editor Keys → New Symmetric Key → Generate**, replace the `k` value with the Base64 secret.
3. Back in Repeater, set `sub` to `administrator`, click **Sign**, choose the key, keep **Don't modify header**.
4. Send `/admin`, then delete `carlos`.

> **Wordlist:** PortSwigger provides a `jwt.secrets.list`; `rockyou` or the `jwt-secrets` list also work.

> **Picture goes here (#7).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-jwt-labs/7.png` _then replace this block with_ `![Lab 3 — JWT authentication bypass via weak signing key](/images/portswigger-jwt-labs/7.png)`_._

## Lab 4 — JWT authentication bypass via jwk header injection
> **Picture goes here (#8).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `jwk`.
> _Save as_ `images/portswigger-jwt-labs/8.png` _then replace this block with_ `![Lab 4 — JWT authentication bypass via jwk header injection](/images/portswigger-jwt-labs/8.png)`_._


**Difficulty:** Practitioner
**Goal:** Embed your own public key in the token and have the server trust it.

If the server reads the verification key from the token's `jwk` header, you control the trust anchor.

1. **JWT Editor Keys → New RSA Key → Generate**.
2. In Repeater set `sub` to `administrator`.
3. Click **Attack → Embedded JWK**, select your RSA key.
4. A `jwk` parameter containing your public key is added to the header. Send `/admin` → access granted.

> **Manual route:** add the `jwk` yourself and set `kid` to match the embedded key's `kid`.

> **Picture goes here (#9).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-jwt-labs/9.png` _then replace this block with_ `![Lab 4 — JWT authentication bypass via jwk header injection](/images/portswigger-jwt-labs/9.png)`_._

## Lab 5 — JWT authentication bypass via jku header injection
> **Picture goes here (#10).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `{`.
> _Save as_ `images/portswigger-jwt-labs/10.png` _then replace this block with_ `![Lab 5 — JWT authentication bypass via jku header injection](/images/portswigger-jwt-labs/10.png)`_._


**Difficulty:** Practitioner
**Goal:** Point the token's `jku` at a JWK Set you host.

`jku` is a URL to a JWK Set. If the server fetches it without validation, host your own.

1. Generate an RSA key in **JWT Editor Keys**.
2. On the exploit server, host a JWK Set containing your public key:

```json
{
  "keys": [
    {
      "kty": "RSA",
      "e": "AQAB",
      "kid": "893d8f0b-061f-42c2-a4aa-5056e12b8ae7",
      "n": "yy1wpYmffgXBxhAUJzHHocCuJolwDqql75ZWuCQ_cb33K2vh9mk6GPM9gNN4Y_qTVX67WhsN3JvaFYw"
    }
  ]
}
```

> **Picture goes here (#11).** Capture this request/response or command step in Burp/terminal. Key line: `{`.
> _Save as_ `images/portswigger-jwt-labs/11.png` _then replace this block with_ `![Lab 5 — JWT authentication bypass via jku header injection](/images/portswigger-jwt-labs/11.png)`_._


3. In the token header, set `kid` to your key's `kid` and add `jku` pointing at the exploit server.
4. Set `sub` to `administrator`, **Sign** with your RSA key (Don't modify header), send `/admin`, delete `carlos`.

> **Picture goes here (#12).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-jwt-labs/12.png` _then replace this block with_ `![Lab 5 — JWT authentication bypass via jku header injection](/images/portswigger-jwt-labs/12.png)`_._

## Lab 6 — JWT authentication bypass via kid header path traversal
> **Picture goes here (#13).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `../../../../../../../dev/null`.
> _Save as_ `images/portswigger-jwt-labs/13.png` _then replace this block with_ `![Lab 6 — JWT authentication bypass via kid header path traversal](/images/portswigger-jwt-labs/13.png)`_._


**Difficulty:** Practitioner
**Goal:** Use `kid` to load a file with predictable contents as the HMAC secret.

If `kid` is used as a filesystem path to look up the key, traverse to a file whose contents you can predict — `/dev/null` (empty).

1. **New Symmetric Key → Generate**, then replace the `k` value with an **empty string**.
2. Set the header `kid` to:

```text
../../../../../../../dev/null
```

> **Picture goes here (#14).** Capture this request/response or command step in Burp/terminal. Key line: `../../../../../../../dev/null`.
> _Save as_ `images/portswigger-jwt-labs/14.png` _then replace this block with_ `![Lab 6 — JWT authentication bypass via kid header path traversal](/images/portswigger-jwt-labs/14.png)`_._


3. Set `sub` to `administrator`, **Sign** with the empty symmetric key (Don't modify header).
4. Send `/admin`, delete `carlos`. The token is effectively signed with an empty secret.

> **Picture goes here (#15).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-jwt-labs/15.png` _then replace this block with_ `![Lab 6 — JWT authentication bypass via kid header path traversal](/images/portswigger-jwt-labs/15.png)`_._

## Lab 7 — Algorithm confusion (RS256 to HS256)
> **Picture goes here (#16).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `/jwks.json`.
> _Save as_ `images/portswigger-jwt-labs/16.png` _then replace this block with_ `![Lab 7 — Algorithm confusion (RS256 to HS256)](/images/portswigger-jwt-labs/16.png)`_._


**Difficulty:** Practitioner
**Goal:** Sign an HS256 token using the server's **public** key as the HMAC secret.

When the server accepts both RS256 and HS256, you can sign with HS256 using the public key (which is not secret).

1. Fetch the public key from `/jwks.json`.
2. In **JWT Editor Keys → New RSA Key**, paste the JWK (JWK option) and save.
3. Right-click the key → **Copy Public Key as PEM**, then Base64-encode the PEM in Decoder.
4. **New Symmetric Key → Generate**, replace `k` with the Base64 PEM.
5. In the token, set `alg` to `HS256`, set `sub` to `administrator`, **Sign** with the symmetric key.
6. Send `/admin`, delete `carlos`.

> **Picture goes here (#17).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-jwt-labs/17.png` _then replace this block with_ `![Lab 7 — Algorithm confusion (RS256 to HS256)](/images/portswigger-jwt-labs/17.png)`_._

## Lab 8 — Algorithm confusion with no exposed key
> **Picture goes here (#18).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `docker run --rm -it portswigger/sig2n <token1> <token2>`.
> _Save as_ `images/portswigger-jwt-labs/18.png` _then replace this block with_ `![Lab 8 — Algorithm confusion with no exposed key](/images/portswigger-jwt-labs/18.png)`_._


**Difficulty:** Expert
**Goal:** Recover the public key from two tokens, then apply the same HS256 confusion.

No JWKS endpoint here, so derive the RSA modulus from signatures.

1. Obtain **two** valid JWTs from the server (log in, copy; log out, log in again, copy).
2. Recover the public key:

```bash
docker run --rm -it portswigger/sig2n <token1> <token2>
```

> **Picture goes here (#19).** Capture this request/response or command step in Burp/terminal. Key line: `docker run --rm -it portswigger/sig2n <token1> <token2>`.
> _Save as_ `images/portswigger-jwt-labs/19.png` _then replace this block with_ `![Lab 8 — Algorithm confusion with no exposed key](/images/portswigger-jwt-labs/19.png)`_._


3. The tool outputs candidate public keys (X.509/PKCS1), each with a tampered JWT. Test each tampered JWT against `/my-account` — a `200` means you found the correct X.509 key (a `302` to `/login` means wrong).
4. Create a **New Symmetric Key** and set `k` to the Base64 X.509 key.
5. Set `alg` to `HS256`, `sub` to `administrator`, **Sign** with that key.
6. Send `/admin`, delete `carlos`.

## JWT attack cheat sheet

| Attack | Condition | Action |
|--------|-----------|--------|
| Unverified signature | Signature not checked | Edit claims, don't re-sign |
| `alg: none` | `none` accepted | Strip signature, keep trailing dot |
| Weak HMAC secret | Guessable key | `hashcat -m 16500`, re-sign |
| `jwk` injection | Key read from header | Embed your public key |
| `jku` injection | Key URL fetched | Host your own JWK Set |
| `kid` path traversal | `kid` used as file path | Point to `/dev/null`, empty key |
| Algorithm confusion | RS256 + HS256 accepted | Sign HS256 with public key |
| Confusion, no key | No JWKS | `sig2n` from two tokens |

## Prevention

- **Always verify the signature** with a single, server-side expected algorithm; never trust the `alg` header.
- Reject `none`, and do not accept a token whose algorithm differs from the one you issued.
- Use **strong, random** HMAC secrets (or asymmetric keys); never weak/guessable strings.
- Ignore `jwk`/`jku`/`kid` values from untrusted tokens — resolve keys from a trusted store, and never treat `kid` as a file path.
- Add short expiries, validate `iss`/`aud`, and rotate keys on compromise.

## Related posts

- [NoSQL Injection labs](/posts/portswigger-nosql-injection-labs/)
- [GraphQL labs](/posts/portswigger-graphql-labs/)
- [API Testing labs](/posts/portswigger-api-testing-labs/)
- [SQL Injection lab series](/posts/portswigger-sqli-part-1-basics/)

> **Picture goes here (#20).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-jwt-labs/20.png` _then replace this block with_ `![Related posts](/images/portswigger-jwt-labs/20.png)`_._

---
title: "PortSwigger GraphQL Labs — Complete Walkthrough"
date: 2026-09-22 08:30:00 +0000
categories: [Web Penetration Testing]
tags: [GraphQL, PortSwigger, Web Security Academy, API Testing]
description: "All 5 PortSwigger GraphQL API labs: introspection for private posts and fields, hidden endpoints with introspection bypass, alias-based brute force, and CSRF."
author: Paschal Sangawe
toc: true
---

GraphQL exposes a single endpoint where clients ask for exactly the data they want. That flexibility is also its weakness: **introspection** can hand an attacker the full schema, fields are often under-authorised, and the ability to send many operations in one request defeats rate limiting. This topic overlaps heavily with API security and IDOR.

This post covers all **5 PortSwigger GraphQL labs**. Burp's built-in **GraphQL** tab and *GraphQL → Set introspection query* / *Save GraphQL queries to site map* actions do most of the heavy lifting.

{% raw %}

## Background

A GraphQL request is usually a `POST` to `/graphql` (or `/api`, `/graphql/v1`) with a JSON body:

```json
{ "query": "{ user(id:1){ username } }" }
```

Introspection asks the API to describe itself:

```graphql
query IntrospectionQuery { __schema { queryType { name } types { ...FullType } } }
```

## Lab 1 — Accessing private GraphQL posts

**Difficulty:** Apprentice
**Goal:** Read a hidden blog post's password.

1. Browse the blog and note posts are fetched via a GraphQL query to `POST /graphql/v1`, each with a sequential `id`. **`id` 3 is missing** → likely a hidden post.
2. Send the request to Repeater and use **GraphQL → Set introspection query**.
3. The schema shows the `BlogPost` type has a **`postPassword`** field.
4. Query the hidden post by adding the field and setting the variable `id` to `3`:

```graphql
query($id: Int!) {
  getBlogPost(id: $id) {
    title
    postPassword
  }
}
```

5. Submit the returned `postPassword`.

> **[Screenshot]** Introspection revealing `postPassword`, then the value for id 3.

## Lab 2 — Accidental exposure of private GraphQL fields

**Difficulty:** Practitioner
**Goal:** Recover the administrator credentials.

1. Log in (attempt) and send the login mutation to Repeater.
2. Run the introspection query. Save queries to the site map (**GraphQL → Save GraphQL queries to site map**).
3. In the schema, find a **`getUser`** query that returns a user's **username and password** by `id`.
4. Send `getUser` to Repeater and iterate the `id` variable — `id: 1` returns the administrator's credentials.
5. Log in as administrator, open the Admin panel, and delete `carlos`.

> **[Screenshot]** The `getUser` query returning the admin username and password.

**Lesson:** GraphQL resolvers must enforce field-level authorization. Returning sensitive fields because the schema exposes them is a classic "accidental exposure" bug.

## Lab 3 — Finding a hidden GraphQL endpoint

**Difficulty:** Practitioner
**Goal:** Discover a non-obvious endpoint, bypass introspection defences, and delete `carlos`.

1. Probe common suffixes. A `GET /api` returns **"Query not present"**, hinting a GraphQL endpoint is there.
2. Send a universal query as a URL parameter:

```text
/api?query=query{__typename}
```

Response:

```json
{ "data": { "__typename": "query" } }
```

3. Run introspection via `?query=...`; it is **disallowed**. The server is filtering queries matching the regex `__schema{`.
4. Bypass it by inserting a **newline** after `__schema` so the regex no longer matches:

```text
/api?query=query+IntrospectionQuery+%7B%0D%0A++__schema%0a+%7B ...
```

Introspection now succeeds.
5. Save queries to the site map. Find `getUser` (iterate `id` to find `carlos` = 3) and the **`deleteOrganizationUser`** mutation.
6. Call the mutation to delete carlos:

```text
/api?query=mutation+%7B%0A%09deleteOrganizationUser%28input%3A%7Bid%3A+3%7D%29+%7B%0A%09%09user+%7B%0A%09%09%09id%0A%09%09%7D%0A%09%7D%0A%7D
```

> **[Screenshot]** The newline introspection bypass and the `deleteOrganizationUser` mutation.

**Lesson:** blocking `__schema{` with a regex is trivially bypassed by whitespace. Disable introspection properly if it isn't needed.

## Lab 4 — Bypassing GraphQL brute force protections

**Difficulty:** Practitioner
**Goal:** Brute-force carlos's password past a rate limiter using aliases.

GraphQL lets one request contain many operations via **aliases**, so a per-request rate limit is meaningless.

1. Attempt logins and observe the API starts returning rate-limit errors.
2. Craft a single `mutation` with many aliased `login` attempts, each a different password for `carlos`:

```graphql
mutation {
  bruteforce0:login(input:{password: "123456", username: "carlos"}) { token success }
  bruteforce1:login(input:{password: "password", username: "carlos"}) { token success }
  ...
  bruteforce99:login(input:{password: "12345678", username: "carlos"}) { token success }
}
```

3. Send it. The response contains each attempt's `success` flag; search for `true` to find the valid password.
4. Log in as `carlos`.

> **[Screenshot]** The aliased mutation batch and the single `success: true` result.

**Lesson:** rate-limit by **operation cost / resolved fields**, not by HTTP request count; disable or limit aliasing and query batching where not needed.

## Lab 5 — Performing CSRF via a GraphQL endpoint

**Difficulty:** Practitioner
**Goal:** Change the victim's email via a cross-site form post.

The GraphQL endpoint accepts form-encoded `POST` requests and relies on cookies without CSRF protection.

1. Log in and change your email; find the GraphQL mutation in history.
2. Send to Repeater and confirm you can change the email again with the session cookie.
3. Convert the request to `POST` with `Content-Type: application/x-www-form-urlencoded` (Burp: *Change request method* twice) and add the URL-encoded body:

```text
query=%0A++++mutation+changeEmail%28%24input%3A+ChangeEmailInput%21%29+%7B%0A++++++++changeEmail%28input%3A+%24input%29+%7B%0A++++++++++++email%0A++++++++%7D%0A++++%7D%0A&operationName=changeEmail&variables=%7B%22input%22%3A%7B%22email%22%3A%22hacker%40hacker.com%22%7D%7D
```

4. Use **Engagement tools → Generate CSRF PoC**, change the target email in the HTML, and deliver it to the victim from the exploit server.

> **[Screenshot]** The form-encoded GraphQL mutation and the generated CSRF PoC.

**Lesson:** GraphQL is not exempt from CSRF. Require a CSRF token or a non-cookie auth mechanism (e.g. `Authorization` header), and reject `application/x-www-form-urlencoded` for GraphQL.

## GraphQL testing cheat sheet

| Goal | Technique |
|------|-----------|
| Discover endpoint | Try `/graphql`, `/api`, `/graphql/v1`, `GET ?query=...` |
| Enumerate schema | Introspection; if blocked, try whitespace/newline/`__schema` variants |
| Find hidden data | Look for fields like `password`, `token`, `postPassword` |
| Bypass IDOR checks | Iterate `id` variables on `getUser`-style queries |
| Defeat rate limits | Alias many mutations into one request |
| CSRF | Convert to form-encoded POST, generate CSRF PoC |
| Discover mutations | Site map → GraphQL queries after introspection |

## Prevention

- Disable **introspection** in production (or restrict it to authenticated admins) and don't rely on naive regex blocks.
- Enforce **field-level and object-level authorization** in every resolver; never return sensitive fields just because they exist.
- Apply **cost/complexity analysis**, depth limits, and disable or cap **aliases/batching** to prevent brute-force and DoS.
- Protect GraphQL like any state-changing endpoint: **CSRF tokens**, `SameSite` cookies, `Authorization` headers, and reject simple form-encoded requests.
- Log and monitor GraphQL operations for anomalous query shapes.

## Related posts

- [API Testing labs](/posts/portswigger-api-testing-labs/)
- [JWT labs](/posts/portswigger-jwt-labs/)
- [NoSQL Injection labs](/posts/portswigger-nosql-injection-labs/)
- [SSRF labs](/posts/portswigger-ssrf-labs/)

{% endraw %}

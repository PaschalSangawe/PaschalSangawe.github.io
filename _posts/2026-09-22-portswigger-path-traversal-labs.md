---
title: "PortSwigger Path Traversal (File Inclusion) Labs — Complete Walkthrough"
date: 2026-09-22 07:00:00 +0000
categories: [Web Penetration Testing]
tags: [Path Traversal, LFI, File Inclusion, PortSwigger, Web Security Academy]
description: "All 6 PortSwigger path traversal / local file inclusion labs, including absolute-path, non-recursive stripping, double URL-decode, start-of-path validation and null byte bypasses."
author: Paschal Sangawe
toc: true
---

Path traversal (also called directory traversal or, in its application-level form, **local file inclusion / LFI**) lets an attacker read files outside the directory the application intends to serve. When user input is concatenated into a filesystem path, `../` sequences walk up the directory tree. A successful hit on `/etc/passwd` proves arbitrary file read; against PHP applications, LFI is often chained to RCE via log poisoning or `php://filter` wrappers.

This post covers all **6 PortSwigger Path Traversal labs**. Every one is the same request (`GET /image?filename=...`) with a different filter to defeat.

## Background — how the sink looks

A typical vulnerable server-side handler:

```php
$file = $_GET['filename'];
echo file_get_contents('/var/www/images/' . $file);
```

Because `$file` is concatenated without normalisation, an attacker controls the path.

## Lab 1 — File path traversal, simple case
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `../../../etc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/1.png` _then replace this block with_ `![Lab 1 — File path traversal, simple case](/images/portswigger-path-traversal-labs/1.png)`_._


**Difficulty:** Apprentice
**Goal:** Retrieve `/etc/passwd`.

No filtering at all.

**Payload** (the `filename` parameter):

```text
../../../etc/passwd
```

> **Picture goes here (#2).** Capture this request/response or command step in Burp/terminal. Key line: `../../../etc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/2.png` _then replace this block with_ `![Lab 1 — File path traversal, simple case](/images/portswigger-path-traversal-labs/2.png)`_._


The path resolves to `/etc/passwd` and its contents are returned.

> **Picture goes here (#3).** The `filename=../../../etc/passwd` request and the `/etc/passwd` contents in the response.
> _Save as_ `images/portswigger-path-traversal-labs/3.png` _then replace this block with_ `![Lab 1 — File path traversal, simple case](/images/portswigger-path-traversal-labs/3.png)`_._

## Lab 2 — Traversal sequences blocked with absolute path bypass
> **Picture goes here (#4).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `/etc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/4.png` _then replace this block with_ `![Lab 2 — Traversal sequences blocked with absolute path bypass](/images/portswigger-path-traversal-labs/4.png)`_._


**Difficulty:** Practitioner
**Goal:** Read `/etc/passwd` when `../` is stripped.

The application blocks traversal sequences but treats the provided filename as relative to a working directory. Because it strips only the literal `../` and does not force the path to stay inside the base directory, we can simply use an **absolute path**:

```text
/etc/passwd
```

> **Picture goes here (#5).** Capture this request/response or command step in Burp/terminal. Key line: `/etc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/5.png` _then replace this block with_ `![Lab 2 — Traversal sequences blocked with absolute path bypass](/images/portswigger-path-traversal-labs/5.png)`_._


> **Picture goes here (#6).** `filename=/etc/passwd` returning the file.
> _Save as_ `images/portswigger-path-traversal-labs/6.png` _then replace this block with_ `![Lab 2 — Traversal sequences blocked with absolute path bypass](/images/portswigger-path-traversal-labs/6.png)`_._

## Lab 3 — Traversal sequences stripped non-recursively
> **Picture goes here (#7).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `....//....//....//etc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/7.png` _then replace this block with_ `![Lab 3 — Traversal sequences stripped non-recursively](/images/portswigger-path-traversal-labs/7.png)`_._


**Difficulty:** Practitioner
**Goal:** Read `/etc/passwd` when the app deletes `../` once.

The filter removes `../` in a single pass but does not loop, so nested sequences survive. Feed it overlapping sequences so that removing the inner `../` leaves a valid `../` behind:

```text
....//....//....//etc/passwd
```

> **Picture goes here (#8).** Capture this request/response or command step in Burp/terminal. Key line: `....//....//....//etc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/8.png` _then replace this block with_ `![Lab 3 — Traversal sequences stripped non-recursively](/images/portswigger-path-traversal-labs/8.png)`_._


The filter matches the `../` in the middle of each `....//`, deletes it, and `....//` collapses to `../`. The path becomes `../../../etc/passwd`.

> **Picture goes here (#9).** The non-recursive bypass payload and the resulting file read.
> _Save as_ `images/portswigger-path-traversal-labs/9.png` _then replace this block with_ `![Lab 3 — Traversal sequences stripped non-recursively](/images/portswigger-path-traversal-labs/9.png)`_._

## Lab 4 — Traversal sequences stripped with superfluous URL-decode
> **Picture goes here (#10).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `..%252f..%252f..%252fetc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/10.png` _then replace this block with_ `![Lab 4 — Traversal sequences stripped with superfluous URL-decode](/images/portswigger-path-traversal-labs/10.png)`_._


**Difficulty:** Practitioner
**Goal:** Read `/etc/passwd` when `../` is stripped and the input is URL-decoded.

The app blocks `../` and then performs an **extra** URL-decode of the input before using it. We exploit the double-decoding by encoding the `%` itself. The slash `%2f` becomes `%252f`:

```text
..%252f..%252f..%252fetc/passwd
```

> **Picture goes here (#11).** Capture this request/response or command step in Burp/terminal. Key line: `..%252f..%252f..%252fetc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/11.png` _then replace this block with_ `![Lab 4 — Traversal sequences stripped with superfluous URL-decode](/images/portswigger-path-traversal-labs/11.png)`_._


The first decode turns `%252f` into `%2f`; the second decode turns `%2f` into `/`, yielding `../../../etc/passwd`.

> **Picture goes here (#12).** Double-encoded payload accepted after the app's own decode.
> _Save as_ `images/portswigger-path-traversal-labs/12.png` _then replace this block with_ `![Lab 4 — Traversal sequences stripped with superfluous URL-decode](/images/portswigger-path-traversal-labs/12.png)`_._

## Lab 5 — Validation of start of path
> **Picture goes here (#13).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `/var/www/images/../../../etc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/13.png` _then replace this block with_ `![Lab 5 — Validation of start of path](/images/portswigger-path-traversal-labs/13.png)`_._


**Difficulty:** Practitioner
**Goal:** Read `/etc/passwd` when the path must start with a fixed folder.

The application transmits the full path and validates that it **starts with** `/var/www/images/`. That check is satisfied by prefixing our traversal with the expected directory:

```text
/var/www/images/../../../etc/passwd
```

> **Picture goes here (#14).** Capture this request/response or command step in Burp/terminal. Key line: `/var/www/images/../../../etc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/14.png` _then replace this block with_ `![Lab 5 — Validation of start of path](/images/portswigger-path-traversal-labs/14.png)`_._


The string starts with the allowed folder, but the `../` sequences still climb out of it.

> **Picture goes here (#15).** The prefixed traversal payload and the file read.
> _Save as_ `images/portswigger-path-traversal-labs/15.png` _then replace this block with_ `![Lab 5 — Validation of start of path](/images/portswigger-path-traversal-labs/15.png)`_._

## Lab 6 — Validation of file extension with null byte bypass
> **Picture goes here (#16).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `../../../etc/passwd%00.png`.
> _Save as_ `images/portswigger-path-traversal-labs/16.png` _then replace this block with_ `![Lab 6 — Validation of file extension with null byte bypass](/images/portswigger-path-traversal-labs/16.png)`_._


**Difficulty:** Practitioner
**Goal:** Read `/etc/passwd` when the filename must end in `.png`.

The app requires the supplied filename to end in `.png`. Appending a URL-encoded **null byte** followed by `.png` satisfies the string check, while the filesystem truncates the path at the null byte and opens the real target:

```text
../../../etc/passwd%00.png
```

> **Picture goes here (#17).** Capture this request/response or command step in Burp/terminal. Key line: `../../../etc/passwd%00.png`.
> _Save as_ `images/portswigger-path-traversal-labs/17.png` _then replace this block with_ `![Lab 6 — Validation of file extension with null byte bypass](/images/portswigger-path-traversal-labs/17.png)`_._


> **Picture goes here (#18).** The `%00.png` payload and the returned `/etc/passwd`.
> _Save as_ `images/portswigger-path-traversal-labs/18.png` _then replace this block with_ `![Lab 6 — Validation of file extension with null byte bypass](/images/portswigger-path-traversal-labs/18.png)`_._

## Bypass cheat sheet

| Defence | Bypass |
|---------|--------|
| None | `../../../etc/passwd` |
| `../` blocked, relative to cwd | Absolute path: `/etc/passwd` |
| `../` stripped once (non-recursive) | `....//....//....//etc/passwd` |
| `../` stripped + extra URL-decode | `..%252f..%252f..%252fetc/passwd` |
| Must start with known folder | `/var/www/images/../../../etc/passwd` |
| Must end with `.png` | `../../../etc/passwd%00.png` |

## Beyond PortSwigger — LFI to RCE

Once you have arbitrary file read in a PHP app, look for:

- **`php://filter/convert.base64-encode/resource=index.php`** — read source code without executing it.
- **Log poisoning** — write PHP into `User-Agent`, then include `/var/log/apache2/access.log`.
- **`/proc/self/environ`**, session files (`/tmp/sess_*`), and uploaded temp files as inclusion sources.
- **PHP session upload progress** (`php://input`) when `session.upload_progress` is enabled.

## Prevention

- Never pass user input directly to filesystem APIs. Resolve the canonical path and verify it stays within an allowed base directory.
- Use an allow-list of permitted filenames/IDs and map them server-side to real paths.
- Strip/normalise traversal sequences **after** all decoding, and reject input containing null bytes.
- Run the application with least privilege so a file read cannot reach sensitive OS files.


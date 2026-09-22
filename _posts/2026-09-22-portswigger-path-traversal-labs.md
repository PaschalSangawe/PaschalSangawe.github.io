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

**Difficulty:** Apprentice
**Goal:** Retrieve `/etc/passwd`.

No filtering at all.

**Payload** (the `filename` parameter):

```text
../../../etc/passwd
```

The path resolves to `/etc/passwd` and its contents are returned.

> **[Screenshot]** The `filename=../../../etc/passwd` request and the `/etc/passwd` contents in the response.

## Lab 2 — Traversal sequences blocked with absolute path bypass

**Difficulty:** Practitioner
**Goal:** Read `/etc/passwd` when `../` is stripped.

The application blocks traversal sequences but treats the provided filename as relative to a working directory. Because it strips only the literal `../` and does not force the path to stay inside the base directory, we can simply use an **absolute path**:

```text
/etc/passwd
```

> **[Screenshot]** `filename=/etc/passwd` returning the file.

## Lab 3 — Traversal sequences stripped non-recursively

**Difficulty:** Practitioner
**Goal:** Read `/etc/passwd` when the app deletes `../` once.

The filter removes `../` in a single pass but does not loop, so nested sequences survive. Feed it overlapping sequences so that removing the inner `../` leaves a valid `../` behind:

```text
....//....//....//etc/passwd
```

The filter matches the `../` in the middle of each `....//`, deletes it, and `....//` collapses to `../`. The path becomes `../../../etc/passwd`.

> **[Screenshot]** The non-recursive bypass payload and the resulting file read.

## Lab 4 — Traversal sequences stripped with superfluous URL-decode

**Difficulty:** Practitioner
**Goal:** Read `/etc/passwd` when `../` is stripped and the input is URL-decoded.

The app blocks `../` and then performs an **extra** URL-decode of the input before using it. We exploit the double-decoding by encoding the `%` itself. The slash `%2f` becomes `%252f`:

```text
..%252f..%252f..%252fetc/passwd
```

The first decode turns `%252f` into `%2f`; the second decode turns `%2f` into `/`, yielding `../../../etc/passwd`.

> **[Screenshot]** Double-encoded payload accepted after the app's own decode.

## Lab 5 — Validation of start of path

**Difficulty:** Practitioner
**Goal:** Read `/etc/passwd` when the path must start with a fixed folder.

The application transmits the full path and validates that it **starts with** `/var/www/images/`. That check is satisfied by prefixing our traversal with the expected directory:

```text
/var/www/images/../../../etc/passwd
```

The string starts with the allowed folder, but the `../` sequences still climb out of it.

> **[Screenshot]** The prefixed traversal payload and the file read.

## Lab 6 — Validation of file extension with null byte bypass

**Difficulty:** Practitioner
**Goal:** Read `/etc/passwd` when the filename must end in `.png`.

The app requires the supplied filename to end in `.png`. Appending a URL-encoded **null byte** followed by `.png` satisfies the string check, while the filesystem truncates the path at the null byte and opens the real target:

```text
../../../etc/passwd%00.png
```

> **[Screenshot]** The `%00.png` payload and the returned `/etc/passwd`.

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


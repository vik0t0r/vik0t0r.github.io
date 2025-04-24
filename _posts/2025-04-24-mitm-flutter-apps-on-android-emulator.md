---
layout: post
title: Mitm flutter apps on android emulator
date: 2025-04-24 15:13 +0200
categories: [Hacking, Software]
tags: [hacking, software, mitm, android,flutter, frida, reverse engineering]
---

## Introduction
Analyzing network traffic from Flutter Android apps can be tricky. In this post, we'll walk through how to intercept and inspect traffic from a Flutter app running on an Android Virtual Device (AVD) using Frida without requiring root access. This is achieved by modifying the APK to include the Frida runtime. Throughout this guide, I will be using Debian as the host OS.

## Step 1: Install android studio, AVD and app
1. Install android studio from [the official source](https://developer.android.com/studio).
2. Create an AVD using the GUI. I recommend selecting an AVD with the Play Store installed for easier APK downloads.
3. Run the AVD and install the app that you want to analyze.

## Step 2: Adding frida to the app
While some automated tools exist, such as [this one](https://github.com/ksg97031/frida-gadget), I’m opting for a manual approach, as I’m not sure how they handle split APKs.

1. Use adb to pull the apk with its splits:
    - Find the package name: `adb shell pm list packages`
    - Find the path for all the splits (usually 4): `adb shell pm path example.com`
    - Pull the APKs (use the paths from the previous command): 
        - `adb pull /data/app/~~XXXXXXXXXXXXXXXX==/example.com-XXXXXXXX==/base.apk`
        - `adb pull /data/app/~~XXXXXXXXXXXXXXXX==/example.com-XXXXXXXX==/split_config.en.apk`
        - `adb pull /data/app/~~XXXXXXXXXXXXXXXX==/example.com-XXXXXXXX==/split_config.x86_64.apk`
        - `adb pull /data/app/~~XXXXXXXXXXXXXXXX==/example.com-XXXXXXXX==/split_config.xxhdpi.apk`

2. Decompile the APKs using [apktool](https://apktool.org/):
    - `apktool d base.apk`
    - `apktool d split_config.x86_64.apk`

3. Add the Frida Gadget to the App:
    - Download the Frida gadget `frida-gadget-XX.X.X-android-x86_64.so.xz` from the [releases page](https://github.com/frida/frida).
    - Unpack the frida gadget `unxz frida-gadget-X.X.XX-android-x86_64.so.xz`.
    - Move the gadget into the lib folder: `mv frida-gadget-X.X.XX-android-x86_64.so split_config.x86_64/lib/x86_64/libfrida-gadget.so`.

3. Patch `base/AndroidManifest.xml`
    - Add the permission for internet access if not already present: `<uses-permission android:name="android.permission.INTERNET"/>`.
    - Set the `android:extractNativeLibs` property to `true`.
    - Locate the entry point for the app, which will have the intent filter for `android.intent.action.MAIN`.

4. Patch the constructor of the main activity:
    - Open the corresponding `.smali` file.
        - Modify the contructor:
        - if `.locals` is 0 change it to 1
        - add the following code to the beginning, after the .locals directive:
        ```
        const-string v0, "frida-gadget"
        invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V
        ```
5. Repack the modified apks:
    - `apktool b base -o base.patched.apk`
    - `apktool b split_config.x86_64 -o split_config.x86_64.patched.apk`

6. Align the modified apks:
    - `zipalign 4 base.patched.apk base.patched.aligned.apk`
    - `zipalign 4 split_config.x86_64.patched.apk split_config.X86_64.patched.aligned.apk`

7. Sign all the apks before installing, I'll be using [this script](https://github.com/vik0t0r/apksigner-debug)
    - `apksign base.patched.aligned.apk`
    - `apksign split_config.x86_64.patched.aligned.apk`
    - `apksign split_config.en.apk`
    - `apksign split_config.xxhdpi.apk`

8. Install the apk
    - `adb install-multiple base.patched.aligned.apk  split_config.x86_64.patched.aligned.apk split_config.en.apk split_config.xxhdpi.apk`

If everything has worked out, launching the app should cause it to freeze at startup. Check Logcat for the following message: `Listening on 127.0.0.1 TCP port 27042` with the tag `Frida`.

## Step 3: Use frida to remove the certificate checks

We will use [this script](https://github.com/NVISOsecurity/disable-flutter-tls-verification) to disable TLS certificate validation. However, on emulators, loaded libraries may not appear with `Process.findModuleByName()`, so we need to disable the check.

Change the following lines:

```js
    var m = Process.findModuleByName(platformConfig["modulename"]);

    if (m === null) {
        console.log('[!] Flutter library not found');
        setTimeout(disableTLSValidation, timeout);
        return;
    }
    else{
        // reset counter so that searching for ssl_verify_peer_cert also gets x attempts
        if(flutterLibraryFound == false){
            flutterLibraryFound = true;
            tries = 1;
        }
    }

```
To this:
```js
flutterLibraryFound = true;
```

## Step 4: Launching the avd with http proxy

1. Start burpsuite (by default listens on `127.0.0.1:8080`).
2. Find the name of the AVD you want to run: `~/Android/Sdk/emulator/emulator -list-avds`.
3. Close the instance on android-studio and launch it from terminal with: `~/Android/Sdk/emulator/emulator -avd name_of_your_avd -http-proxy http://127.0.0.1:8080`.

Now, launch Frida with the path to the modified script: `frida -U Gadget -l modified_script.js`

If everything works correctly, you should now see the app's traffic in cleartext in Burp Suite.

## Conclusion
In this post, we successfully intercepted and analyzed network traffic from a Flutter app on an Android emulator using Frida. This approach, which doesn't require root access, enables effective traffic analysis and can be used for further security testing and reverse engineering.

## References
- [https://koz.io/using-frida-on-android-without-root/](https://koz.io/using-frida-on-android-without-root/)
- [https://blog.nviso.eu/2022/08/18/intercept-flutter-traffic-on-ios-and-android-http-https-dio-pinning/](https://koz.io/using-frida-on-android-without-root/)
- [https://github.com/NVISOsecurity/disable-flutter-tls-verification](https://koz.io/using-frida-on-android-without-root/)
- [https://blog.mindedsecurity.com/2024/05/bypassing-certificate-pinning-on.html](https://koz.io/using-frida-on-android-without-root/)


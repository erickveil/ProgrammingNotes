---
title: How To Verify the SDK Level Your APK Targets
layout: note
tags:
  - android
---

The Google Play Store has just sent you that email that your Android App needs to target the latest SDK Level or it's going to get pulled from the store. You go in to your manifest and make the change, build the APK, and you're ready to deliver it...

But wait. Are you *sure* this APK is on the right API level? How do you test for that?

First, make sure java is installed.
In a git command line, run `java --version`

1. Use the GitHub repo to install apktool [https://ibotpeaches.github.io/Apktool/](https://ibotpeaches.github.io/Apktool/).
2. After installing then run the following command -> `java -jar apktool.jar d file.apk`
3. open the file folder.
4. locate the file named `apktool.yml`
5. the information is found in the SDK info
## Setting the target SDK

Disassembling a build of CSR's apk, it shows the following in apktool.yml:

```
dkInfo:
  minSdkVersion: '23'
  targetSdkVersion: '30'
```

We want to target SDK version 33.
The targeted version is possibly set in the `pro` file:

```
ANDROID_TARGET_SDK_VERSION = 30
```

While the minimum is possbly set in the Jenkins `Compile.sh` file:

```
export ANDROID_NDK_ROOT=/Users/devops/Library/Android/sdk/ndk/23.1.7779620
```

The numbers match the compiled apk's results.

After changing the value in the `pro` file to `33`, the output of the apktool.yml file reflects the change:

```
sdkInfo:
  minSdkVersion: '23'
  targetSdkVersion: '33'
```

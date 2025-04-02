# React Native OTP Auto-Reading with `react-native-otp-verify` (Fix for Play Store Downloads)

This guide explains how to set up OTP auto-reading using the `react-native-otp-verify` library and how to fix the issue where OTP auto-reading works in release mode but not when downloaded from the Play Store.


https://www.npmjs.com/package/react-native-otp-verify

## Table of Contents
- [Why Does OTP Auto-Reading Fail After Downloading from Play Store?](#why-does-otp-auto-reading-fail-after-downloading-from-play-store)
- [Solution: Fixing the SHA1 Hash Key Issue](#solution-fixing-the-sha1-hash-key-issue)
- [Step-by-Step Guide to Fix the Issue](#step-by-step-guide-to-fix-the-issue)
  - [1. Get the Correct SHA1 Certificate from Play Store](#1-get-the-correct-sha1-certificate-from-play-store)
  - [2. Generate the Correct Hash Key](#2-generate-the-correct-hash-key)
  - [3. Update the OTP SMS Format](#3-update-the-otp-sms-format)
- [Important Considerations](#important-considerations)
  - [When Will the Hash Key Change?](#when-will-the-hash-key-change)
- [Final Thoughts](#final-thoughts)

## Why Does OTP Auto-Reading Fail After Downloading from Play Store?

When your app is downloaded from the **Play Store**, the **hash key** used in the OTP SMS verification changes because Google Play signs your APK with a different signing key compared to your local development environment. This change causes the **hash key** mismatch between the SMS and your app, preventing OTP auto-reading.

## Solution: Fixing the SHA1 Hash Key Issue

The root cause of the issue is that the **SHA1 certificate** used to sign your app for release in the Play Store is different from the one used in your local development environment. To resolve this, you need to use the **SHA1 certificate** from Google Play and generate a new hash key.


## you can use this to get hash key to set in sms 
so you have to use adb commnad :

```
$ adb logcat |  grep "APPApp hash:"
04-02 12:36:04.211  9467  9527 I ReactNativeJS: 'APPApp hash:', [ 'xtU+/Q+OYAf' ]

```

in this `APPApp hash:` is a console.log key to see in log 

```
  useEffect(() => {
    getHash()
      .then(async hash => {
        console.log('APPApp hash:', hash)
        console.log("check res", res)
      })
      .catch(console.warn);

  }, []);
```

than you will get proper hask key which you can use.


### Step-by-Step Guide to Fix the Issue

### 1. Get the Correct SHA1 Certificate from Play Store

1. Go to the **Google Play Console** → Select Your App.
2. Navigate to **Setup** → **App Integrity**.
3. Copy the **SHA1 fingerprint** listed under **App Signing Certificate**.

### 2. Generate the Correct Hash Key

Once you have the **SHA1 fingerprint**, you can generate the correct hash key by running the following command. This step will ensure that the hash key in the OTP matches the one used by the Play Store version of your app.

**PowerShell Command (Windows)**:
```powershell
$hashKey = echo -n "com.satyafinlead DE:F0:95:0E:E1:E0:86:F3:FB:5A:40:1D:3B:84:EA:90:0F:46:AC:24" | openssl sha256 -binary | openssl base64
$hashKey.Substring(0,11)
```

This will output the hash key that you need to include in the OTP SMS format.
For example, the output might look like:


```
dWs/P2Q/Px1

```

### 3. Update the OTP SMS Format
Once you have the correct hash key, make sure your OTP SMS follows this format:

```
Your OTP is 123456. Do not share it.
dWs/P2Q/Px1
```

This key must be added to the SMS message that is sent to the user, which will then be read by the app for automatic OTP retrieval.

# Important Considerations
### When Will the Hash Key Change?
The hash key will not change automatically unless:

1. The package name (e.g., com.satyafinleadgether) is changed.

2. Your SHA1 certificate changes. This can happen if you:

 - Change your app's signing key.

 - Google Play changes your signing key (rare, but possible if you reset it).

If the package name and signing key remain the same, the hash key will remain the same.


## Example Commands Summary:

1. PowerShell Command to Generate the Correct Hash Key:

```
    $hashKey = echo -n "com.satyafinlead DE:F0:95:0E:E1:E0:86:F3:FB:5A:40:1D:3B:84:EA:90:0F:46:AC:24" | openssl sha256 -binary | openssl base64
$hashKey.Substring(0,11)

```

# MirrorCast TV Receiver — APK Build Guide

The receiver is a single HTML file. Here are 3 ways to turn it into an
installable Android TV APK, from easiest to most advanced.

---

## ✅ METHOD 1 — Median.co (Free, No coding, 5 minutes)

This is the recommended method. Median wraps your HTML into a real APK.

1. Go to https://median.co
2. Click "Get Started" → choose Android
3. When asked for your app URL, select "Upload HTML file"
4. Upload `receiver.html` from this folder
5. Set these options:
   - App Name: MirrorCast TV
   - Package ID: com.mirrorcast.tv
   - Orientation: Landscape
   - Keep screen on: YES
   - Hide navigation bar: YES (fullscreen)
6. Click Build → Download APK
7. Done — install on TV using ADB or USB (see below)

Cost: Free for one app

---

## ✅ METHOD 2 — Android Studio (Free, Full control)

If you want a proper signed APK you own completely:

### Prerequisites
- Download Android Studio: https://developer.android.com/studio
- Install it (takes ~10 mins)

### Steps

1. Open Android Studio → New Project → "No Activity" template
2. Set:
   - Name: MirrorCast TV
   - Package: com.mirrorcast.tv
   - Language: Java
   - Min SDK: API 21 (Android 5.0)
3. Copy `receiver.html` into: `app/src/main/assets/receiver.html`
4. Replace the contents of `app/src/main/res/layout/activity_main.xml` with:

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <WebView
        android:id="@+id/webview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</FrameLayout>
```

5. Create `app/src/main/java/com/mirrorcast/tv/MainActivity.java`:

```java
package com.mirrorcast.tv;

import android.app.Activity;
import android.os.Bundle;
import android.webkit.WebSettings;
import android.webkit.WebView;
import android.webkit.WebChromeClient;
import android.webkit.PermissionRequest;
import android.view.View;

public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Fullscreen
        getWindow().getDecorView().setSystemUiVisibility(
            View.SYSTEM_UI_FLAG_FULLSCREEN |
            View.SYSTEM_UI_FLAG_HIDE_NAVIGATION |
            View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY
        );

        setContentView(R.layout.activity_main);
        WebView wv = findViewById(R.id.webview);

        WebSettings ws = wv.getSettings();
        ws.setJavaScriptEnabled(true);
        ws.setMediaPlaybackRequiresUserGesture(false);
        ws.setDomStorageEnabled(true);           // Lets the TV remember saved PCs
        ws.setDatabaseEnabled(true);
        ws.setAllowFileAccessFromFileURLs(true);
        ws.setAllowUniversalAccessFromFileURLs(true);
        ws.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
        ws.setCacheMode(WebSettings.LOAD_DEFAULT);

        // Grant camera/mic/WebRTC permissions automatically
        wv.setWebChromeClient(new WebChromeClient() {
            @Override
            public void onPermissionRequest(PermissionRequest request) {
                request.grant(request.getResources());
            }
        });

        // Keep screen on
        wv.setKeepScreenOn(true);

        // Load the receiver from assets
        wv.loadUrl("file:///android_asset/receiver.html");
    }
}
```

6. Update `app/src/main/AndroidManifest.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.mirrorcast.tv">

    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
    <uses-feature android:name="android.hardware.touchscreen" android:required="false"/>
    <uses-feature android:name="android.software.leanback" android:required="false"/>

    <application
        android:label="MirrorCast TV"
        android:icon="@mipmap/ic_launcher"
        android:theme="@style/Theme.AppCompat.NoActionBar"
        android:banner="@drawable/banner">

        <activity
            android:name=".MainActivity"
            android:configChanges="orientation|screenSize|keyboardHidden"
            android:screenOrientation="landscape"
            android:launchMode="singleTask">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <!-- Makes it appear on Android TV launcher -->
                <category android:name="android.intent.category.LEANBACK_LAUNCHER"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

7. Create a banner image for the TV launcher:
   - Size: 320x180 pixels
   - Save as: `app/src/main/res/drawable/banner.png`
   - Can be any simple image (the TV home screen shows this)

8. Build: **Build → Build Bundle(s)/APK(s) → Build APK(s)**
9. APK will be at: `app/build/outputs/apk/debug/app-debug.apk`

---

## METHOD 3 — Capacitor CLI (For developers)

If you have Node.js installed:

```bash
npm install -g @capacitor/cli
npx cap init "MirrorCast TV" com.mirrorcast.tv
npx cap add android

# Copy receiver.html into www/
mkdir -p www
cp receiver.html www/index.html

# Open in Android Studio to build
npx cap open android
```

---

## Installing the APK on the TV

### Via ADB (WiFi — no USB needed)

```bash
# 1. Enable on TV: Settings → About → click Build 7 times → Developer Options → USB Debugging ON
# 2. Find TV IP: Settings → Network → WiFi → your network → IP address

# 3. On your PC (open Command Prompt):
adb connect 192.168.1.XXX:5555
# TV will show a prompt — select ALLOW with remote

adb install mirrorcast-tv.apk
# Done! Find "MirrorCast TV" in your apps list
```

### Via USB Drive

```bash
# 1. Copy APK to USB drive (FAT32)
# 2. Plug into TV
# 3. TV Settings → Apps → Unknown Sources: ON (or Security → Install Unknown Apps)
# 4. Open file manager on TV (or install FX File Explorer from Play Store)
# 5. Browse to USB → tap mirrorcast-tv.apk → Install
```

---

## Notes

- The app uses `localStorage` to remember saved PCs — this persists across app restarts
- WebRTC works natively in Android WebView (API 21+)
- The app requests fullscreen + keeps screen on automatically
- If the TV shows a permission dialog for network access, select Allow

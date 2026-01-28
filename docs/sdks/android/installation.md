---
title: Installation
description: Install the Clippr Android SDK in your project
---

# Installation

Add the Clippr SDK to your Android project.

## Maven Central

Add the dependency to your app's `build.gradle.kts`:

```kotlin
dependencies {
    implementation("xyz.useclippr:clippr:0.0.4")
}
```

Or with Groovy `build.gradle`:

```groovy
dependencies {
    implementation 'xyz.useclippr:clippr:0.0.4'
}
```

## JitPack (Alternative)

Add JitPack repository to your `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://jitpack.io") }
    }
}
```

Then add the dependency:

```kotlin
dependencies {
    implementation("com.github.nexlabstudio:clippr-android:0.0.4")
}
```

## Sync and Build

After adding the dependency, sync your project:

1. Click **Sync Now** in the Gradle notification, or
2. **File** → **Sync Project with Gradle Files**

## Configure App Links

### Add Intent Filter

Add to your `AndroidManifest.xml`:

```xml
<manifest ...>
    <application ...>
        <activity
            android:name=".MainActivity"
            android:launchMode="singleTop"
            android:exported="true">

            <!-- Your existing intent filters -->

            <!-- Clippr App Links -->
            <intent-filter android:autoVerify="true">
                <action android:name="android.intent.action.VIEW" />
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data
                    android:scheme="https"
                    android:host="yourapp.clppr.xyz" />
            </intent-filter>

        </activity>
    </application>
</manifest>
```

Replace `yourapp.clppr.xyz` with your subdomain from the dashboard.

### Set Launch Mode

Use `singleTop` or `singleTask` to properly handle links when the app is already open:

```xml
<activity
    android:name=".MainActivity"
    android:launchMode="singleTop"
    ...>
```

## Add SHA256 Fingerprints

App Links require SHA256 fingerprints for verification.

### Get Debug Fingerprint

```bash
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```

### Get Release Fingerprint

```bash
keytool -list -v -keystore /path/to/release.keystore -alias your-alias
```

### Google Play App Signing

If using Play App Signing:

1. Go to **Google Play Console** → Your App → **Setup** → **App signing**
2. Copy the **SHA256** from "App signing key certificate"
3. Add both your upload key AND Play's signing key to the Clippr dashboard

## Verify Installation

```kotlin
import xyz.useclippr.sdk.Clippr

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        Clippr.initialize(
            context = this,
            apiKey = "YOUR_API_KEY",
            debug = true // Enable debug logs
        )
    }
}
```

With `debug = true`, you'll see initialization logs in Logcat.

## Verify Asset Links

Clippr hosts your Asset Links file. Verify it:

```bash
curl https://yourapp.clppr.xyz/.well-known/assetlinks.json
```

Expected response:

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.yourcompany.yourapp",
      "sha256_cert_fingerprints": ["..."]
    }
  }
]
```

## Troubleshooting

### App Links Not Verified

Check verification status:

```bash
adb shell pm get-app-links com.yourcompany.yourapp
```

If not verified:
1. Ensure SHA256 fingerprints are correct in dashboard
2. Check assetlinks.json is accessible
3. Clear app data and reinstall

### Dependency Conflicts

If you have OkHttp conflicts:

```kotlin
implementation("xyz.useclippr:clippr:0.0.4") {
    exclude(group = "com.squareup.okhttp3", module = "okhttp")
}
```

### ProGuard/R8 Rules

The SDK includes ProGuard rules automatically. If needed manually:

```proguard
-keep class xyz.useclippr.sdk.** { *; }
```

## Next Steps

- [Quick Start](/sdks/android/quickstart) - Initialize and handle links
- [App Links](/sdks/android/app-links) - Deep dive into App Links

---
title: Installation
description: Install the Clippr Flutter SDK in your project
---

# Installation

Add the Clippr SDK to your Flutter project.

## Add Dependency

Add `clippr` to your `pubspec.yaml`:

```yaml
dependencies:
  clippr: ^0.0.4
```

Then run:

```bash
flutter pub get
```

## Platform Setup

### iOS Setup

#### 1. Add Associated Domains

In Xcode:

1. Open your iOS project (`ios/Runner.xcworkspace`)
2. Select your target → **Signing & Capabilities**
3. Click **+ Capability** and add **Associated Domains**
4. Add: `applinks:yourapp.clppr.xyz`

Replace `yourapp` with your app's subdomain from the Clippr dashboard.

#### 2. Handle Universal Links (Optional)

If you're using a custom `AppDelegate`, add Universal Link handling:

```swift
// ios/Runner/AppDelegate.swift
import UIKit
import Flutter

@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GeneratedPluginRegistrant.register(with: self)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  // Handle Universal Links
  override func application(
    _ application: UIApplication,
    continue userActivity: NSUserActivity,
    restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void
  ) -> Bool {
    return super.application(application, continue: userActivity, restorationHandler: restorationHandler)
  }
}
```

### Android Setup

#### 1. Add Intent Filter

Add the intent filter to your `AndroidManifest.xml`:

```xml
<!-- android/app/src/main/AndroidManifest.xml -->
<manifest ...>
  <application ...>
    <activity
      android:name=".MainActivity"
      android:launchMode="singleTop"
      ...>

      <!-- Existing intent filters -->

      <!-- Add this intent filter for Clippr deep links -->
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

Replace `yourapp.clppr.xyz` with your app's subdomain.

#### 2. Configure Launch Mode

Ensure your MainActivity uses `singleTop` or `singleTask` launch mode to properly handle links when the app is already open:

```xml
<activity
  android:name=".MainActivity"
  android:launchMode="singleTop"
  ...>
```

## Verify Installation

Run your app and check the logs for successful initialization:

```dart
import 'package:clippr/clippr.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Clippr.initialize(
    apiKey: 'YOUR_API_KEY',
    debug: true, // Enable debug logging
  );

  runApp(MyApp());
}
```

With `debug: true`, you should see logs like:

```
[Clippr] Initialized with API key: YOUR_***_KEY
[Clippr] Checking for deferred link...
```

## Troubleshooting

### iOS: Associated Domains not working

1. Verify the entitlements file exists at `ios/Runner/Runner.entitlements`
2. Check that the Associated Domains capability is enabled in your Apple Developer account
3. Verify the AASA file is accessible: `curl https://yourapp.clppr.xyz/.well-known/apple-app-site-association`

### Android: App Links not verified

1. Verify the assetlinks.json is accessible: `curl https://yourapp.clppr.xyz/.well-known/assetlinks.json`
2. Ensure your SHA256 fingerprint is configured correctly in the Clippr dashboard
3. Test verification: `adb shell pm get-app-links com.yourcompany.yourapp`

### Build errors

If you encounter build errors, try:

```bash
flutter clean
flutter pub get
cd ios && pod install && cd ..
flutter run
```

## Next Steps

- [Quick Start](/sdks/flutter/quickstart) - Initialize and handle your first deep link
- [Handling Links](/sdks/flutter/handling-links) - Learn about link handling patterns

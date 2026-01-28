---
title: Installation
description: Install the Clippr iOS SDK in your project
---

# Installation

Add the Clippr SDK to your iOS project using Swift Package Manager or CocoaPods.

## Swift Package Manager (Recommended)

### Using Xcode

1. Open your project in Xcode
2. Go to **File** → **Add Packages...**
3. Enter the repository URL:
   ```
   https://github.com/nexlabstudio/clippr-ios.git
   ```
4. Select version `0.0.4` or later
5. Click **Add Package**

### Using Package.swift

Add the dependency to your `Package.swift`:

```swift
dependencies: [
    .package(url: "https://github.com/nexlabstudio/clippr-ios.git", from: "0.0.4")
]
```

Then add `ClipprSDK` to your target:

```swift
targets: [
    .target(
        name: "YourApp",
        dependencies: ["ClipprSDK"]
    )
]
```

## CocoaPods

Add to your `Podfile`:

```ruby
pod 'ClipprSDK', '~> 0.0.4'
```

Then run:

```bash
pod install
```

## Verify Installation

Import the SDK and initialize it:

```swift
import ClipprSDK

// In your App init or AppDelegate
Clippr.initialize(apiKey: "YOUR_API_KEY", debug: true)
```

With `debug: true`, you should see initialization logs in the console.

## Platform Configuration

### Add Associated Domains

Universal Links require the Associated Domains capability:

1. In Xcode, select your target
2. Go to **Signing & Capabilities**
3. Click **+ Capability**
4. Add **Associated Domains**
5. Add your domain:
   ```
   applinks:yourapp.clppr.xyz
   ```

Replace `yourapp` with your app's subdomain from the Clippr dashboard.

### Verify AASA File

Clippr automatically hosts your Apple App Site Association (AASA) file. Verify it's accessible:

```bash
curl https://yourapp.clppr.xyz/.well-known/apple-app-site-association
```

Expected response:

```json
{
  "applinks": {
    "apps": [],
    "details": [
      {
        "appID": "TEAMID.com.yourcompany.yourapp",
        "paths": ["*"]
      }
    ]
  }
}
```

### App Tracking Transparency (Optional)

If you want to use IDFA for deterministic matching, add the usage description to your `Info.plist`:

```xml
<key>NSUserTrackingUsageDescription</key>
<string>We use this identifier to improve your experience and measure campaign effectiveness.</string>
```

The SDK will automatically request tracking permission and use the IDFA if granted.

## Troubleshooting

### Package Resolution Failed

If SPM fails to resolve the package:

1. Try **File** → **Packages** → **Reset Package Caches**
2. Clean build folder: **Product** → **Clean Build Folder**
3. Restart Xcode

### Associated Domains Not Working

1. Verify your Team ID is correct in the Clippr dashboard
2. Check the AASA file is accessible (see above)
3. Ensure the Associated Domains capability is enabled in your Apple Developer account
4. Delete the app and reinstall (Associated Domains are cached)

### Linker Errors with CocoaPods

If you see linker errors:

```bash
pod deintegrate
pod install
```

Then clean and rebuild.

## Next Steps

- [Quick Start](/sdks/ios/quickstart) - Initialize and handle your first deep link
- [Universal Links](/sdks/ios/universal-links) - Deep dive into Universal Links setup

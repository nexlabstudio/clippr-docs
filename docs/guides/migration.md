---
title: Firebase Dynamic Links Migration
description: Migrate from Firebase Dynamic Links to Clippr
draft: true
---

<!-- This page is intentionally unpublished. Clippr positioning is
attribution-first, not FDL replacement. Kept around in case the team wants
to relight the migration angle for a specific campaign later. The Stardust
nav entry is also commented out. -->

# Firebase Dynamic Links Migration

Firebase Dynamic Links is deprecated and will shut down on August 25, 2025. This guide helps you migrate to Clippr with minimal disruption.

## Why Migrate to Clippr?

- **Similar API**: Designed to feel familiar to Firebase users
- **Better attribution**: Install Referrer support for 100% accuracy on Android
- **Modern architecture**: Built for scale and reliability
- **Active development**: New features and improvements
- **Fair pricing**: Generous free tier and transparent pricing

## Migration Overview

```
1. Create Clippr account and app
2. Update SDK dependencies
3. Update code (minimal changes)
4. Update Universal Links / App Links configuration
5. Create new links in Clippr
6. Redirect old links (optional)
7. Test and deploy
```

## Step 1: Create Clippr Account

1. Sign up at [app.useclippr.xyz](https://app.useclippr.xyz)
2. Create an organization
3. Create an app with your iOS/Android configuration
4. Copy your API key

## Step 2: Update Dependencies

<Tabs>
<Tab name="Flutter">

**Remove Firebase:**
```yaml
# pubspec.yaml - Remove these
dependencies:
  firebase_dynamic_links: ^5.0.0  # Remove
```

**Add Clippr:**
```yaml
dependencies:
  clippr: ^0.0.4
```

</Tab>
<Tab name="iOS">

**Remove Firebase:**
```ruby
# Podfile - Remove these
pod 'FirebaseDynamicLinks'
```

**Add Clippr:**
```ruby
pod 'ClipprSDK', '~> 0.0.4'
```

Or via SPM, remove FirebaseDynamicLinks and add:
```
https://github.com/nexlabstudio/clippr-ios.git
```

</Tab>
<Tab name="Android">

**Remove Firebase:**
```kotlin
// build.gradle.kts - Remove
implementation("com.google.firebase:firebase-dynamic-links-ktx")
```

**Add Clippr:**
```kotlin
implementation("xyz.useclippr:clippr:0.0.4")
```

</Tab>
</Tabs>

## Step 3: Update Code

### Initialization

<Tabs>
<Tab name="Flutter">

**Before (Firebase):**
```dart
import 'package:firebase_dynamic_links/firebase_dynamic_links.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MyApp());
}
```

**After (Clippr):**
```dart
import 'package:clippr/clippr.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Clippr.initialize(apiKey: 'YOUR_API_KEY');
  runApp(MyApp());
}
```

</Tab>
<Tab name="iOS">

**Before (Firebase):**
```swift
import FirebaseDynamicLinks

FirebaseApp.configure()
```

**After (Clippr):**
```swift
import ClipprSDK

Clippr.initialize(apiKey: "YOUR_API_KEY")
```

</Tab>
<Tab name="Android">

**Before (Firebase):**
```kotlin
// Firebase initializes automatically
```

**After (Clippr):**
```kotlin
import xyz.useclippr.sdk.Clippr

class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        Clippr.initialize(this, "YOUR_API_KEY")
    }
}
```

</Tab>
</Tabs>

### Getting Initial Link

<Tabs>
<Tab name="Flutter">

**Before:**
```dart
final PendingDynamicLinkData? data =
    await FirebaseDynamicLinks.instance.getInitialLink();
if (data != null) {
  final Uri deepLink = data.link;
  // Handle link
}
```

**After:**
```dart
final link = await Clippr.getInitialLink();
if (link != null) {
  final path = link.path; // e.g., "/product/123"
  // Handle link
}
```

</Tab>
<Tab name="iOS">

**Before:**
```swift
DynamicLinks.dynamicLinks().handleUniversalLink(url) { link, error in
    guard let link = link else { return }
    // Handle link
}
```

**After:**
```swift
if let link = await Clippr.getInitialLink() {
    let path = link.path
    // Handle link
}
```

</Tab>
<Tab name="Android">

**Before:**
```kotlin
Firebase.dynamicLinks.getDynamicLink(intent)
    .addOnSuccessListener { pendingData ->
        val deepLink = pendingData?.link
        // Handle link
    }
```

**After:**
```kotlin
val link = Clippr.getInitialLink()
if (link != null) {
    val path = link.path
    // Handle link
}
```

</Tab>
</Tabs>

### Listening for Links

<Tabs>
<Tab name="Flutter">

**Before:**
```dart
FirebaseDynamicLinks.instance.onLink.listen((dynamicLinkData) {
  final Uri deepLink = dynamicLinkData.link;
});
```

**After:**
```dart
Clippr.onLink = (link) {
  final path = link.path;
};
```

</Tab>
<Tab name="iOS">

**After:**
```swift
Clippr.onLink = { link in
    let path = link.path
}
```

</Tab>
<Tab name="Android">

**After:**
```kotlin
Clippr.onLink = { link ->
    val path = link.path
}
```

</Tab>
</Tabs>

### Creating Links

<Tabs>
<Tab name="Flutter">

**Before:**
```dart
final dynamicLinkParams = DynamicLinkParameters(
  link: Uri.parse('https://example.com/product/123'),
  uriPrefix: 'https://example.page.link',
  androidParameters: AndroidParameters(packageName: 'com.example.app'),
  iosParameters: IOSParameters(bundleId: 'com.example.app'),
);
final link = await FirebaseDynamicLinks.instance.buildShortLink(dynamicLinkParams);
```

**After:**
```dart
final params = LinkParameters(
  path: '/product/123',
  campaign: 'my_campaign',
);
final shortLink = await Clippr.createLink(params);
// shortLink.url contains the short URL
```

</Tab>
</Tabs>

## Step 4: Update Platform Configuration

### iOS - Associated Domains

**Before:**
```
applinks:example.page.link
```

**After:**
```
applinks:yourapp.clppr.xyz
```

### Android - Intent Filter

**Before:**
```xml
<data android:host="example.page.link" android:scheme="https"/>
```

**After:**
```xml
<data android:host="yourapp.clppr.xyz" android:scheme="https"/>
```

## Step 5: Migrate Links

### Option A: Recreate Links

For most cases, recreate your links in Clippr:

1. Export your Firebase links
2. Create equivalent links in Clippr dashboard
3. Update any hardcoded links in marketing materials

### Option B: Redirect Old Links

If you have many links in circulation:

1. Set up a redirect from your Firebase domain
2. Redirect to equivalent Clippr links
3. Gradually phase out old links

## Step 6: Testing

1. **Test direct links**: Click link with app installed
2. **Test deferred links**: Uninstall, click link, install, open
3. **Test attribution**: Verify campaign/source/medium data
4. **Test all platforms**: iOS, Android, web fallback

## API Comparison Table

| Firebase Dynamic Links | Clippr | Notes |
|------------------------|--------|-------|
| `getInitialLink()` | `getInitialLink()` | Same concept |
| `onLink` stream | `onLink` callback | Similar pattern |
| `DynamicLinkParameters` | `LinkParameters` | Simpler structure |
| `buildShortLink()` | `createLink()` | Same result |
| `link.link` (Uri) | `link.path` (String) | Path only, simpler |

## Data Model Changes

### Firebase PendingDynamicLinkData

```dart
// Firebase
data.link              // Full URI
data.utmParameters     // Map
```

### Clippr ClipprLink

```dart
// Clippr
link.path              // String path
link.attribution       // campaign, source, medium
link.metadata          // Custom data
link.matchType         // How it was matched
```

## Timeline

1. **Now**: Start migration, run both in parallel
2. **Before Aug 2025**: Complete migration
3. **Aug 25, 2025**: Firebase Dynamic Links shuts down

## Need Help?

- [Clippr Documentation](/)
- [GitHub Issues](https://github.com/nexlabstudio/clippr-flutter/issues)
- [FAQ](/resources/faq)

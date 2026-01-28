---
title: Troubleshooting
description: Solutions to common Clippr issues
---

# Troubleshooting

Solutions to common issues with Clippr integration.

## Deep Linking Issues

### Links Open in Browser Instead of App

**iOS:**

1. Verify Associated Domains:
   - Open Xcode → Target → Signing & Capabilities
   - Check `applinks:yourapp.clppr.xyz` is listed

2. Verify AASA file:
   ```bash
   curl https://yourapp.clppr.xyz/.well-known/apple-app-site-association
   ```

3. Check Team ID matches:
   ```json
   {"applinks":{"details":[{"appID":"TEAMID.com.your.bundleid"}]}}
   ```

4. Delete and reinstall app (clears cached Associated Domains)

**Android:**

1. Verify intent filter:
   ```xml
   <intent-filter android:autoVerify="true">
       <data android:scheme="https" android:host="yourapp.clppr.xyz"/>
   </intent-filter>
   ```

2. Check verification status:
   ```bash
   adb shell pm get-app-links com.your.package
   ```

3. Re-verify:
   ```bash
   adb shell pm verify-app-links --re-verify com.your.package
   ```

4. Verify assetlinks.json and SHA256 fingerprint

### getInitialLink() Returns Null

**Possible causes:**

1. **SDK not initialized**: Ensure `Clippr.initialize()` is called before `getInitialLink()`

2. **Already retrieved**: `getInitialLink()` only returns data once per install for deferred links

3. **No link clicked**: User installed organically without clicking a link

4. **Attribution window expired**: Click was more than 72 hours ago

5. **Different device**: User clicked on one device, installed on another

**Debug steps:**

```dart
await Clippr.initialize(apiKey: 'KEY', debug: true);
// Check console logs for matching details
final link = await Clippr.getInitialLink();
print('Link: $link');
```

### Low Match Rate

**Causes:**

1. Long time between click and install
2. VPN or proxy usage changing IP
3. Users opting out of tracking
4. Shared devices

**Improvements:**

1. Initialize SDK as early as possible
2. Request ATT permission on iOS
3. Use Install Referrer on Android (automatic)
4. Create links closer to conversion (shorter window)

## SDK Issues

### Initialization Fails

**Check:**

1. Valid API key from dashboard
2. Internet connectivity
3. Correct SDK version

```dart
try {
  await Clippr.initialize(apiKey: 'YOUR_KEY', debug: true);
} catch (e) {
  print('Init failed: $e');
}
```

### Events Not Tracking

1. Verify SDK is initialized
2. Check for errors in completion handler
3. Verify internet connectivity
4. Check rate limits (500/min for events)

```dart
try {
  await Clippr.track('event_name');
  print('Tracked successfully');
} catch (e) {
  print('Track failed: $e');
}
```

### Links Not Creating

1. Check API key permissions
2. Verify alias isn't already taken
3. Check rate limits (30/min for link creation)

```dart
try {
  final link = await Clippr.createLink(params);
  print('Created: ${link.url}');
} catch (e) {
  print('Create failed: $e');
}
```

## Platform-Specific Issues

### iOS: AASA Not Loading

1. **Check URL directly:**
   ```bash
   curl -v https://yourapp.clppr.xyz/.well-known/apple-app-site-association
   ```

2. **Verify content type**: Should be `application/json`

3. **Check for redirects**: AASA must be served directly, not redirected

4. **Use Apple's validator**: [search.developer.apple.com/appsearch-validation-tool](https://search.developer.apple.com/appsearch-validation-tool)

### iOS: Tracking Permission Issues

ATT must be requested before Clippr can access IDFA:

```swift
import AppTrackingTransparency

ATTrackingManager.requestTrackingAuthorization { status in
    switch status {
    case .authorized:
        print("Tracking authorized")
    case .denied, .restricted:
        print("Tracking denied - using fingerprinting")
    case .notDetermined:
        print("Not determined")
    @unknown default:
        break
    }

    // Initialize Clippr after permission is determined
    Clippr.initialize(apiKey: "KEY")
}
```

### Android: Install Referrer Not Working

1. **Verify Play Store install**: Install Referrer only works for Play Store installs

2. **Check dependency**:
   ```kotlin
   implementation("com.android.installreferrer:installreferrer:2.2")
   ```

3. **Test with actual Play Store**: Debug installs from Android Studio don't have referrer data

### Android: SHA256 Mismatch

**Get correct fingerprint:**

```bash
# Debug
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android

# Release
keytool -list -v -keystore /path/to/release.keystore -alias your-alias
```

**If using Play App Signing:**
- Get fingerprint from Play Console → Setup → App signing
- Add BOTH Play's signing key AND your upload key

### Flutter: Platform Channel Errors

1. **Clean and rebuild:**
   ```bash
   flutter clean
   flutter pub get
   cd ios && pod install && cd ..
   flutter run
   ```

2. **Check Swift/Kotlin versions** match SDK requirements

3. **Verify native project configuration**

## Dashboard Issues

### Links Not Showing

1. Check you're in the correct organization
2. Verify app filter is correct
3. Refresh the page

### Analytics Not Updating

1. Analytics may have up to 5-minute delay
2. Verify SDK is sending events
3. Check time zone settings

### API Key Not Working

1. Verify key is for correct app
2. Check key hasn't been rotated
3. Ensure using correct header: `X-API-Key` or `Authorization: Bearer`

## Getting Help

If issues persist:

1. **Enable debug mode** to get detailed logs
2. **Check GitHub issues** for similar problems
3. **Open a new issue** with:
   - SDK version
   - Platform and OS version
   - Steps to reproduce
   - Logs and error messages

### GitHub Repositories

- [Flutter SDK](https://github.com/nexlabstudio/clippr-flutter)
- [iOS SDK](https://github.com/nexlabstudio/clippr-ios-sdk)
- [Android SDK](https://github.com/nexlabstudio/clippr-android-sdk)

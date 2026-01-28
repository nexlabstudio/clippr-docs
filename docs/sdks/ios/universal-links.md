---
title: Universal Links
description: Configure Universal Links for your iOS app
---

# Universal Links

Universal Links allow users to tap a link and open your app directly, without going through Safari first.

## How Universal Links Work

1. Apple verifies your domain ownership via the AASA file
2. When the user taps a link to your domain, iOS checks if an app is associated
3. If associated, the app opens directly; if not, Safari opens

```
User taps: https://yourapp.clppr.xyz/product/123
         ↓
iOS checks: Is there an app for yourapp.clppr.xyz?
         ↓
    ┌────────────────┐     ┌────────────────┐
    │  App Installed │     │ Not Installed  │
    │       ↓        │     │       ↓        │
    │  Opens app to  │     │  Opens Safari  │
    │  /product/123  │     │  (then store)  │
    └────────────────┘     └────────────────┘
```

## Configuration

### 1. Add Associated Domains Capability

In Xcode:

1. Select your project in the navigator
2. Select your app target
3. Go to **Signing & Capabilities** tab
4. Click **+ Capability**
5. Add **Associated Domains**
6. Add your domain:

```
applinks:yourapp.clppr.xyz
```

<Warning>
Replace `yourapp` with your actual subdomain from the Clippr dashboard.
</Warning>

### 2. Verify Your Team ID

Your Team ID must match what's configured in Clippr:

1. Go to [developer.apple.com](https://developer.apple.com)
2. Navigate to **Membership**
3. Copy your **Team ID**
4. Ensure it's entered correctly in the Clippr dashboard app settings

### 3. Verify AASA File

Clippr hosts your Apple App Site Association file automatically. Verify it:

```bash
curl -v https://yourapp.clppr.xyz/.well-known/apple-app-site-association
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

The `appID` format is: `{TeamID}.{BundleID}`

## Handling Universal Links

### SwiftUI (iOS 14+)

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
                .onOpenURL { url in
                    // Handles both Universal Links and custom schemes
                    Clippr.handleUniversalLink(url)
                }
        }
    }
}
```

### SceneDelegate (iOS 13+)

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    // Handle links from cold start
    func scene(
        _ scene: UIScene,
        willConnectTo session: UISceneSession,
        options connectionOptions: UIScene.ConnectionOptions
    ) {
        // Check for Universal Link in connection options
        if let userActivity = connectionOptions.userActivities.first(
            where: { $0.activityType == NSUserActivityTypeBrowsingWeb }
        ) {
            Clippr.handleUniversalLink(userActivity)
        }
    }

    // Handle links while app is running (background → foreground)
    func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
        Clippr.handleUniversalLink(userActivity)
    }

    // Handle URL scheme links
    func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        if let url = URLContexts.first?.url {
            Clippr.handleUniversalLink(url)
        }
    }
}
```

### AppDelegate (Legacy)

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {

    // Handle Universal Links
    func application(
        _ application: UIApplication,
        continue userActivity: NSUserActivity,
        restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void
    ) -> Bool {
        return Clippr.handleUniversalLink(userActivity)
    }

    // Handle URL scheme
    func application(
        _ app: UIApplication,
        open url: URL,
        options: [UIApplication.OpenURLOptionsKey: Any] = [:]
    ) -> Bool {
        return Clippr.handleUniversalLink(url)
    }
}
```

## Testing Universal Links

### On Device

1. **Don't tap links in Safari**: Safari may not trigger Universal Links for the same domain
2. **Use Notes or Messages**: Type/paste the link and tap it
3. **Long press**: Shows "Open in [App Name]" if configured correctly

### Debug Checklist

If Universal Links aren't working:

1. **Check Associated Domains**
   - Open Xcode → Target → Signing & Capabilities
   - Verify `applinks:yourapp.clppr.xyz` is listed

2. **Verify AASA is accessible**
   ```bash
   curl https://yourapp.clppr.xyz/.well-known/apple-app-site-association
   ```

3. **Check Team ID and Bundle ID**
   - AASA shows `appID` as `TEAMID.bundleid`
   - Both must match your app exactly

4. **Delete and reinstall the app**
   - Associated Domains are cached aggressively
   - Reinstalling forces a refresh

5. **Check system logs**
   ```bash
   # On Mac with device connected
   log stream --predicate 'subsystem == "com.apple.AppSSO"' --debug
   ```

### Using Apple's Validator

Apple provides a validator tool:

1. Go to [search.developer.apple.com/appsearch-validation-tool/](https://search.developer.apple.com/appsearch-validation-tool/)
2. Enter your domain: `yourapp.clppr.xyz`
3. Check for errors in the AASA file

## Common Issues

### "Open in Safari" Instead of App

**Possible causes:**
- Associated Domains not configured correctly
- AASA file not accessible or malformed
- Team ID mismatch
- App not installed via App Store/TestFlight (for production)

**Solutions:**
- Delete and reinstall the app
- Verify AASA file contents
- Check Xcode entitlements

### Links Work Sometimes

**Possible cause:** User has disabled Universal Links for your app.

**Solution:** Long-press the link → "Open in [App Name]" resets the preference.

### Works in Development, Not Production

**Possible causes:**
- Different Bundle ID for debug/release
- Team ID not matching
- AASA cached from earlier incorrect version

**Solutions:**
- Ensure Bundle IDs match between builds
- Wait 24-48 hours for AASA cache to refresh
- Test with a new device that hasn't seen the old AASA

## Custom Domains

If you're using a custom domain instead of `clppr.xyz`:

1. Configure the custom domain in Clippr dashboard
2. Update Associated Domains:
   ```
   applinks:links.yourdomain.com
   ```
3. Clippr will host the AASA at your custom domain

See [Custom Domains Guide](/guides/custom-domains) for setup instructions.

## Next Steps

- [Handling Links](/sdks/ios/handling-links) - Process deep links in your app
- [Creating Links](/sdks/ios/creating-links) - Generate short links
- [Troubleshooting](/resources/troubleshooting) - More debugging tips

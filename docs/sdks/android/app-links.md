---
title: App Links
description: Configure Android App Links for your app
---

# App Links

Android App Links allow users to tap a link and open your app directly, without a disambiguation dialog.

## How App Links Work

1. Android verifies domain ownership via Digital Asset Links
2. When user taps a verified link, your app opens directly
3. If verification fails or app not installed, browser opens

```
User taps: https://yourapp.clppr.xyz/product/123
         ↓
Android checks: Is this domain verified for this app?
         ↓
    ┌────────────────┐     ┌────────────────┐
    │    Verified    │     │  Not Verified  │
    │       ↓        │     │       ↓        │
    │   Opens app    │     │ Shows chooser  │
    │   directly     │     │   or browser   │
    └────────────────┘     └────────────────┘
```

## Configuration

### 1. Add Intent Filter

In your `AndroidManifest.xml`:

```xml
<activity
    android:name=".MainActivity"
    android:launchMode="singleTop"
    android:exported="true">

    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />

        <data
            android:scheme="https"
            android:host="yourapp.clppr.xyz" />
    </intent-filter>

</activity>
```

Key points:
- `android:autoVerify="true"` triggers automatic verification
- Use `https` scheme (required for App Links)
- Replace `yourapp.clppr.xyz` with your subdomain

### 2. Configure SHA256 Fingerprints

Get your signing certificate fingerprints and add them to the Clippr dashboard.

#### Debug Keystore

```bash
keytool -list -v \
  -keystore ~/.android/debug.keystore \
  -alias androiddebugkey \
  -storepass android \
  -keypass android
```

#### Release Keystore

```bash
keytool -list -v \
  -keystore /path/to/your/release.keystore \
  -alias your-key-alias
```

#### Google Play App Signing

If you use Play App Signing (recommended):

1. Go to **Google Play Console**
2. Navigate to **Setup** → **App signing**
3. Copy both:
   - **App signing key** SHA256 (used by Play Store)
   - **Upload key** SHA256 (your local signing key)

Add BOTH fingerprints to the Clippr dashboard.

### 3. Verify Asset Links

Clippr automatically hosts your `assetlinks.json`. Verify:

```bash
curl https://yourapp.clppr.xyz/.well-known/assetlinks.json
```

Expected:

```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.yourcompany.yourapp",
      "sha256_cert_fingerprints": [
        "AB:CD:EF:...:12:34"
      ]
    }
  }
]
```

## Handling App Links

### In MainActivity

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Handle the App Link that launched the activity
        handleIntent(intent)
    }

    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        // Handle links when activity is already running
        handleIntent(intent)
    }

    private fun handleIntent(intent: Intent?) {
        // Let Clippr SDK handle the link
        Clippr.handle(intent)
    }
}
```

### With Navigation Component

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var navController: NavController

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        navController = navHostFragment.navController

        Clippr.handle(intent)

        lifecycleScope.launch {
            Clippr.getInitialLink()?.let { link ->
                navigateToDeepLink(link.path)
            }
        }

        Clippr.onLink = { link ->
            navigateToDeepLink(link.path)
        }
    }

    private fun navigateToDeepLink(path: String) {
        // Convert Clippr path to Navigation deep link
        val uri = Uri.parse("yourapp://$path")
        navController.navigate(uri)
    }
}
```

## Testing App Links

### Check Verification Status

```bash
adb shell pm get-app-links com.yourcompany.yourapp
```

Output should show:

```
com.yourcompany.yourapp:
    ID: ...
    Signatures: [...]
    Domain verification state:
      yourapp.clppr.xyz: verified
```

### Test with ADB

```bash
adb shell am start \
  -a android.intent.action.VIEW \
  -d "https://yourapp.clppr.xyz/test/path" \
  com.yourcompany.yourapp
```

### Manual Testing

1. Send link via messaging app (not from browser)
2. Tap the link
3. App should open directly (no app chooser)

## Troubleshooting

### App Chooser Still Appears

**Possible causes:**
- Verification failed
- Missing `android:autoVerify="true"`
- Incorrect SHA256 fingerprint
- User manually chose browser before

**Solutions:**

1. Check verification status with `adb shell pm get-app-links`

2. Re-verify manually:
   ```bash
   adb shell pm verify-app-links --re-verify com.yourcompany.yourapp
   ```

3. Clear app defaults:
   - Settings → Apps → Your App → Open by default → Clear defaults

### Verification Always Fails

1. **Check assetlinks.json is accessible:**
   ```bash
   curl -I https://yourapp.clppr.xyz/.well-known/assetlinks.json
   ```
   Should return `200 OK` with `Content-Type: application/json`

2. **Verify fingerprint format:**
   - Must be uppercase
   - Colon-separated
   - 64 characters (SHA256)

3. **Check package name matches exactly**

### Works in Debug, Not Release

You likely have different signing keys:
- Add BOTH debug and release fingerprints
- If using Play App Signing, add the Play signing key fingerprint

## Multiple Domains

For custom domains or multiple subdomains:

```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />

    <!-- Default Clippr domain -->
    <data
        android:scheme="https"
        android:host="yourapp.clppr.xyz" />

    <!-- Custom domain -->
    <data
        android:scheme="https"
        android:host="links.yourdomain.com" />
</intent-filter>
```

Each domain needs its own assetlinks.json file.

## Next Steps

- [Handling Links](/sdks/android/handling-links) - Process links in your app
- [Creating Links](/sdks/android/creating-links) - Generate short links
- [Custom Domains](/guides/custom-domains) - Use your own domain

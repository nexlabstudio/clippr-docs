---
title: Managing Apps
description: Create and configure apps in the Clippr dashboard
---

# Managing Apps

Apps in Clippr represent your mobile applications. Each app gets its own subdomain, API key, and configuration.

## Create an App

1. Navigate to **Apps** in the sidebar
2. Click **Create App**
3. Fill in the details:

| Field | Description | Example |
|-------|-------------|---------|
| **Name** | Display name | My Awesome App |
| **Subdomain** | Unique identifier for links | `myapp` → `myapp.clppr.xyz` |

## iOS Configuration

Configure these settings for iOS Universal Links:

| Field | Description | Where to Find |
|-------|-------------|---------------|
| **Bundle ID** | App's bundle identifier | Xcode → Target → General |
| **App Store ID** | Numeric ID from App Store URL | `apps.apple.com/app/id123456789` |
| **Team ID** | Apple Developer Team ID | developer.apple.com → Membership |

### Finding Your Team ID

1. Go to [developer.apple.com](https://developer.apple.com)
2. Sign in and go to **Membership**
3. Copy your **Team ID**

## Android Configuration

Configure these settings for Android App Links:

| Field | Description | Where to Find |
|-------|-------------|---------------|
| **Package Name** | App's package name | `build.gradle` → `applicationId` |
| **SHA256 Fingerprints** | Signing certificate fingerprints | See below |

### Getting SHA256 Fingerprints

**Debug keystore:**
```bash
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android
```

**Release keystore:**
```bash
keytool -list -v -keystore /path/to/release.keystore -alias your-alias
```

**Google Play App Signing:**
1. Go to Play Console → Your App → Setup → App signing
2. Copy SHA256 from "App signing key certificate"
3. Also add your upload key fingerprint

<Warning>
If using Play App Signing, add BOTH fingerprints (Play's signing key + your upload key).
</Warning>

## API Key

Each app has a unique API key for SDK authentication.

### Viewing Your API Key

1. Go to **Apps** → Select your app
2. The API key is displayed in the app details

### Rotating Your API Key

If your key is compromised:

1. Click **Rotate API Key**
2. Confirm the rotation
3. Update your app and release an update

<Warning>
Rotating immediately invalidates the old key. Users on old app versions will lose attribution until they update.
</Warning>

## Verification Files

Clippr automatically hosts verification files:

- **iOS AASA**: `https://yourapp.clppr.xyz/.well-known/apple-app-site-association`
- **Android Asset Links**: `https://yourapp.clppr.xyz/.well-known/assetlinks.json`

### Verify Configuration

```bash
# iOS
curl https://yourapp.clppr.xyz/.well-known/apple-app-site-association

# Android
curl https://yourapp.clppr.xyz/.well-known/assetlinks.json
```

## Edit an App

1. Go to **Apps** → Select your app
2. Click **Edit**
3. Update the fields
4. Click **Save**

## Delete an App

<Warning>
Deleting an app removes all associated links and analytics data. This cannot be undone.
</Warning>

1. Go to **Apps** → Select your app
2. Click **Delete**
3. Confirm deletion

## Next Steps

- [Creating Links](/dashboard/links) - Create deep links for your app
- [Analytics](/dashboard/analytics) - Track app performance

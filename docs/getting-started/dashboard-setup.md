---
title: Dashboard Setup
description: Set up your organization and apps in the Clippr dashboard
---

# Dashboard Setup

This guide walks you through setting up your Clippr account, creating organizations, and configuring apps.

## Create Your Account

1. Go to [app.useclippr.xyz](https://app.useclippr.xyz)
2. Click **Sign Up**
3. Enter your email and create a password
4. Verify your email address

<Info>
You can also sign up with Google or GitHub for faster onboarding.
</Info>

## Create an Organization

Organizations are workspaces that contain your apps, links, and team members.

<Steps>

### Click "Create Organization"
After signing in, you'll be prompted to create your first organization.

### Enter organization details
- **Name**: Your company or project name
- **Slug**: URL-friendly identifier (e.g., `my-company`)

### Choose a plan
Select a plan based on your needs:

| Plan | Apps | Links | Team Members |
|------|------|-------|--------------|
| **Starter** | 1 | 1,000 | 1 |
| **Growth** | 5 | 10,000 | 5 |
| **Pro** | 20 | 100,000 | 20 |
| **Enterprise** | Unlimited | Unlimited | Unlimited |

</Steps>

## Create an App

Apps represent your mobile applications. Each app gets its own subdomain and API key.

<Steps>

### Navigate to Apps
Click **Apps** in the sidebar, then **Create App**.

### Enter app details

| Field | Description | Example |
|-------|-------------|---------|
| **Name** | Display name for your app | My Awesome App |
| **Subdomain** | Unique subdomain for your links | `myapp` → `myapp.clppr.xyz` |

### Configure iOS settings (if applicable)

| Field | Description | Where to Find |
|-------|-------------|---------------|
| **Bundle ID** | Your app's bundle identifier | Xcode → Target → General |
| **App Store ID** | Numeric ID from App Store URL | `apps.apple.com/app/id123456789` → `123456789` |
| **Team ID** | Apple Developer Team ID | [developer.apple.com](https://developer.apple.com) → Membership |

### Configure Android settings (if applicable)

| Field | Description | Where to Find |
|-------|-------------|---------------|
| **Package Name** | Your app's package name | `build.gradle` → `applicationId` |
| **SHA256 Fingerprints** | Signing certificate fingerprints | See below |

### Save and copy your API key
After creating the app, copy the API key. You'll need it for SDK initialization.

</Steps>

## Getting Android SHA256 Fingerprints

Android requires SHA256 fingerprints for App Links verification.

### Debug Keystore

```bash
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```

### Release Keystore

```bash
keytool -list -v -keystore /path/to/your/release.keystore -alias your-alias
```

### Google Play App Signing

If using Google Play App Signing:
1. Go to Google Play Console → Your App → Setup → App signing
2. Copy the SHA256 fingerprint from "App signing key certificate"

<Warning>
Add both your upload key fingerprint AND Google Play's app signing fingerprint if you use Play App Signing.
</Warning>

## Verify Your Configuration

After creating your app, Clippr automatically hosts:

- **iOS AASA**: `https://yourapp.clppr.xyz/.well-known/apple-app-site-association`
- **Android Asset Links**: `https://yourapp.clppr.xyz/.well-known/assetlinks.json`

You can verify these by visiting the URLs in your browser.

### Test AASA (iOS)

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

### Test Asset Links (Android)

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

## API Key Management

### Viewing Your API Key

1. Go to **Apps** → Select your app
2. The API key is shown in the app details

### Rotating Your API Key

If your API key is compromised:

1. Go to **Apps** → Select your app
2. Click **Rotate API Key**
3. Confirm the rotation
4. Update your app with the new key and release an update

<Warning>
Rotating the API key immediately invalidates the old key. Make sure to update your app before rotating in production.
</Warning>

## Team Management

Invite team members to collaborate on your organization.

### Invite a Member

1. Go to **Organization Settings** → **Members**
2. Click **Invite Member**
3. Enter their email address
4. Select their role:
   - **Owner**: Full access, can manage billing and delete org
   - **Admin**: Can manage apps, links, and members
   - **Member**: Can create and manage links
   - **Viewer**: Read-only access

### Remove a Member

1. Go to **Organization Settings** → **Members**
2. Click the menu next to the member
3. Click **Remove**

## Next Steps

Your dashboard is now configured! Next:

- [Quick Start](/getting-started/quickstart) - Integrate the SDK
- [Creating Links](/dashboard/links) - Create your first deep link
- [Analytics](/dashboard/analytics) - Track your link performance

---
title: Custom Domains
description: Use your own domain for Clippr short links
---

# Custom Domains

Use your own domain instead of `clppr.xyz` for branded short links.

## Overview

**Default:**
```
https://myapp.clppr.xyz/summer-sale
```

**With custom domain:**
```
https://links.mycompany.com/summer-sale
```

## Requirements

- Growth plan or higher
- Domain you own
- Access to DNS settings

## Setup Process

### Step 1: Add Domain in Dashboard

1. Go to **Apps** → Select your app
2. Click **Custom Domains** → **Add Domain**
3. Enter your domain: `links.yourcompany.com`
4. Click **Add**

### Step 2: Configure DNS

Add a CNAME record pointing to Clippr:

| Type | Name | Value |
|------|------|-------|
| CNAME | links | `custom.clppr.xyz` |

**Example for common providers:**

**Cloudflare:**
1. Go to DNS settings
2. Add record:
   - Type: CNAME
   - Name: links
   - Target: custom.clppr.xyz
   - Proxy status: DNS only (gray cloud)

**GoDaddy:**
1. Go to DNS Management
2. Add record:
   - Type: CNAME
   - Host: links
   - Points to: custom.clppr.xyz

**Namecheap:**
1. Go to Advanced DNS
2. Add new record:
   - Type: CNAME
   - Host: links
   - Value: custom.clppr.xyz

### Step 3: Verify Domain

1. Return to dashboard
2. Click **Verify** next to your domain
3. Wait for DNS propagation (can take up to 48 hours)

### Step 4: SSL Certificate

Clippr automatically provisions an SSL certificate via Let's Encrypt. This happens after DNS verification.

## Update Platform Configuration

### iOS - Associated Domains

Add your custom domain to Associated Domains:

```
applinks:links.yourcompany.com
```

Keep the original domain too for backwards compatibility:

```
applinks:myapp.clppr.xyz
applinks:links.yourcompany.com
```

### Android - Intent Filter

Add your custom domain to the intent filter:

```xml
<intent-filter android:autoVerify="true">
    <action android:name="android.intent.action.VIEW" />
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />

    <!-- Original domain -->
    <data android:scheme="https" android:host="myapp.clppr.xyz" />

    <!-- Custom domain -->
    <data android:scheme="https" android:host="links.yourcompany.com" />
</intent-filter>
```

## Verification Files

Clippr hosts verification files on your custom domain:

- `https://links.yourcompany.com/.well-known/apple-app-site-association`
- `https://links.yourcompany.com/.well-known/assetlinks.json`

## Using Custom Domain

Once verified, create links with your custom domain:

**Dashboard:**
- New links automatically use custom domain
- Existing links continue to work on both domains

**SDK:**
```dart
final params = LinkParameters(path: '/product/123');
final link = await Clippr.createLink(params);
// Returns: https://links.yourcompany.com/abc123
```

## Multiple Domains

You can add multiple custom domains:

- `links.yourcompany.com` - Main brand
- `go.yourbrand.com` - Marketing
- `share.yourapp.io` - User sharing

Each domain can be assigned to different apps.

## Removing a Domain

1. Go to **Custom Domains**
2. Click **Remove** next to the domain
3. Confirm removal

<Warning>
Links on the removed domain will stop working immediately.
</Warning>

## Troubleshooting

### DNS Not Propagating

- Wait up to 48 hours
- Check with: `dig links.yourcompany.com CNAME`
- Verify no conflicting records

### SSL Certificate Failed

- Ensure DNS is correctly configured
- Check for CAA records that might block Let's Encrypt
- Contact support if persists after 24 hours

### Universal Links Not Working

1. Verify AASA is accessible:
   ```bash
   curl https://links.yourcompany.com/.well-known/apple-app-site-association
   ```
2. Check Team ID in response matches your app
3. Delete and reinstall app to refresh Associated Domains

### App Links Not Verified

1. Verify assetlinks.json:
   ```bash
   curl https://links.yourcompany.com/.well-known/assetlinks.json
   ```
2. Check SHA256 fingerprint matches
3. Re-verify: `adb shell pm verify-app-links --re-verify your.package.name`

## Next Steps

- [Creating Links](/dashboard/links) - Create links with your custom domain
- [Universal Links](/sdks/ios/universal-links) - iOS deep linking setup
- [App Links](/sdks/android/app-links) - Android deep linking setup

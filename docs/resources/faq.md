---
title: FAQ
description: Frequently asked questions about Clippr
---

# Frequently Asked Questions

## General

### What is Clippr?

Clippr is a deep linking and mobile attribution platform. It helps you:
- Create short links that open your app directly
- Track where your users come from
- Measure marketing campaign effectiveness
- Handle deferred deep linking (attribution after install)

<!-- FAQ entry disabled — Clippr positioning is attribution-first, not FDL replacement.
### Is Clippr a replacement for Firebase Dynamic Links?

Yes. Clippr is designed as a modern replacement for Firebase Dynamic Links, which is being deprecated. The API is intentionally similar for easy migration.
-->

### What platforms does Clippr support?

- iOS (13.0+)
- Android (API 21+)
- Flutter (3.10+)
- Web (fallback URLs)

### Is Clippr free?

Clippr offers a free Starter plan with:
- 1 app
- 1,000 links
- 30 days analytics retention

Paid plans offer more apps, links, custom domains, and extended analytics.

## Deep Linking

### What's the difference between deep linking and deferred deep linking?

**Deep linking**: Opens your app directly to specific content when the app is already installed.

**Deferred deep linking**: Remembers the link content even when the app isn't installed, and delivers it after the user installs and opens the app.

### How does deferred deep linking work?

1. User clicks a link
2. Clippr records the click with a device fingerprint
3. User is redirected to the app store
4. User installs and opens the app
5. The SDK sends the device fingerprint to Clippr
6. Clippr matches it to the original click
7. The link data is returned to your app

### What is the match rate?

Match rate varies by method:
- Install Referrer (Android): 100%
- IDFA/GAID matching: 100%
- Probabilistic/fingerprint: 85-95%

Overall match rates of 70-90% are typical.

### How long do deferred links stay valid?

The default attribution window is 72 hours. Clicks older than this have lower match accuracy.

## SDKs

### Do I need to use the SDK?

For full functionality (deferred deep linking, event tracking), yes. For simple link redirects, links work without any SDK.

### Can I use Clippr with React Native?

Currently we offer Flutter, iOS, and Android SDKs. For React Native, you can:
1. Use the native SDKs via native modules
2. Use the REST API directly
3. Request React Native SDK support via GitHub issues

### How do I test deferred deep linking?

1. Create a test link in the dashboard
2. Uninstall your app
3. Click the link (you'll be redirected to the store)
4. Install the app via Xcode/Android Studio
5. Open the app
6. The SDK should return the link data

### Why isn't getInitialLink() returning data?

Common causes:
- SDK not initialized before calling
- Link was already retrieved (it's one-time)
- More than 72 hours since click
- User installed from different device than clicked

## Universal Links & App Links

### Why aren't Universal Links working on iOS?

1. Check Associated Domains capability is enabled
2. Verify the domain format: `applinks:yourapp.clppr.xyz`
3. Verify AASA file: `curl https://yourapp.clppr.xyz/.well-known/apple-app-site-association`
4. Delete and reinstall the app (Associated Domains are cached)

### Why aren't App Links working on Android?

1. Verify intent filter has `android:autoVerify="true"`
2. Check assetlinks.json: `curl https://yourapp.clppr.xyz/.well-known/assetlinks.json`
3. Verify SHA256 fingerprint is correct
4. Check status: `adb shell pm get-app-links com.your.package`

### Can I use a custom domain?

Yes, on Growth plan and higher. See the [Custom Domains guide](/guides/custom-domains).

## Attribution

### What attribution data can I track?

- Campaign name
- Traffic source
- Marketing medium
- Custom metadata

### How do I see attribution in the dashboard?

Go to Analytics → filter by campaign, source, or medium to see performance breakdowns.

### Does Clippr support multi-touch attribution?

Currently, Clippr uses last-touch attribution (the last click before install gets credit).

## Privacy & Compliance

### Is Clippr GDPR compliant?

Yes. Clippr:
- Only collects data necessary for attribution
- Supports user data deletion requests
- Provides data processing agreements
- Allows disabling fingerprinting

### Does Clippr use cookies?

No. Clippr uses device fingerprinting and install referrer data, not cookies.

### How does Clippr work with App Tracking Transparency (ATT)?

Clippr respects ATT status:
- If user opts in: Uses IDFA for deterministic matching
- If user opts out: Uses probabilistic fingerprinting (lower accuracy)

### Can users opt out of tracking?

Users can:
- Disable tracking at OS level (ATT on iOS, Ad ID on Android)
- Request data deletion through your app

## Billing

### How are links counted?

Each unique link created counts toward your limit. Clicks don't count against the limit.

### What happens if I exceed my link limit?

You'll receive warnings as you approach the limit. At the limit, you won't be able to create new links until:
- You upgrade your plan
- The next billing period starts

### Can I change plans?

Yes. Upgrades take effect immediately (prorated). Downgrades take effect at the next billing cycle.

## Support

### How do I get help?

- Documentation: You're here!
- GitHub Issues: For bugs and feature requests
- Email: support@useclippr.xyz

### How do I report a bug?

Open an issue on the relevant GitHub repository:
- [Flutter SDK](https://github.com/nexlabstudio/clippr-flutter/issues)
- [iOS SDK](https://github.com/nexlabstudio/clippr-ios-sdk/issues)
- [Android SDK](https://github.com/nexlabstudio/clippr-android-sdk/issues)

Include:
- SDK version
- Platform and OS version
- Steps to reproduce
- Expected vs. actual behavior

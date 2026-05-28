---
title: Introduction
description: Welcome to Clippr - Deep linking and mobile attribution for iOS, Android, and Flutter
---

# Welcome to Clippr

Clippr is a modern deep linking and mobile attribution platform for iOS, Android, and Flutter. Create beautiful short links, track user attribution, and deliver users directly to content within your app.

## Why Clippr?

<Cards>
  <Card title="Deep Linking" icon="link">
    Create short links that open your app directly to the right content, with automatic fallbacks to app stores or web.
  </Card>
  <Card title="Deferred Deep Linking" icon="clock">
    Attribute users even when your app isn't installed. Users are routed to the correct content after installation.
  </Card>
  <Card title="Attribution & Analytics" icon="chart">
    Track campaigns, sources, and user journeys with comprehensive analytics and real-time dashboards.
  </Card>
  <Card title="Cross-Platform SDKs" icon="devices">
    Native SDKs for iOS, Android, and Flutter with consistent APIs and easy integration.
  </Card>
</Cards>

## Quick Links

| Resource | Description |
|----------|-------------|
| [Quick Start](/getting-started/quickstart) | Get up and running in 5 minutes |
| [Flutter SDK](/sdks/flutter) | Integrate Clippr in your Flutter app |
| [iOS SDK](/sdks/ios) | Integrate Clippr in your iOS app |
| [Android SDK](/sdks/android) | Integrate Clippr in your Android app |
| [API Reference](/api) | REST API documentation |

## How It Works

```
1. Create a short link in the Clippr dashboard
   → https://yourapp.clppr.xyz/summer-sale

2. User clicks the link
   → If app installed: Opens directly to content
   → If not installed: Redirects to app store

3. User opens app (even after installation)
   → SDK retrieves the original deep link
   → Your app navigates to the right content

4. Track everything
   → Clicks, installs, and conversions
   → Campaign attribution
   → Revenue tracking
```

## Features

### Universal Links & App Links
Automatic configuration for iOS Universal Links and Android App Links. No manual AASA or assetlinks.json hosting required.

### Deferred Deep Linking
Users who install your app after clicking a link are still routed to the correct content. Works via deterministic matching (Install Referrer on Android) and probabilistic fingerprinting.

### Campaign Attribution
Track where your users come from with UTM-style parameters: campaign, source, and medium. See which marketing channels drive the most installs and revenue.

### Short Link Creation
Create branded short links from the dashboard or programmatically via SDK/API. Customize with aliases, social preview metadata, and expiration dates.

### Real-time Analytics
Monitor clicks, installs, and match rates in real-time. Identify your top-performing links and campaigns.

### SKAdNetwork Support
Full support for Apple's SKAdNetwork for privacy-preserving attribution on iOS 14.5+.

## Get Started

Ready to integrate Clippr? Start with our [Quick Start Guide](/getting-started/quickstart) to create your first deep link in under 5 minutes.

<Steps>

### Create an account
Sign up at [app.useclippr.xyz](https://app.useclippr.xyz) and create your organization.

### Create an app
Register your mobile app with its iOS Bundle ID and/or Android package name.

### Integrate the SDK
Add the Clippr SDK to your Flutter, iOS, or Android project.

### Create your first link
Use the dashboard to create a short link and test it on your device.

</Steps>

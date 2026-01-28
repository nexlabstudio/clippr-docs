---
title: SDK Overview
description: Choose the right Clippr SDK for your mobile platform
---

# SDK Overview

Clippr provides native SDKs for Flutter, iOS, and Android. All SDKs share a consistent API design for easy cross-platform development.

## Available SDKs

<Cards>
  <Card title="Flutter SDK" icon="flutter" href="/sdks/flutter">
    Cross-platform SDK for Flutter apps. Supports iOS and Android with a single codebase.
  </Card>
  <Card title="iOS SDK" icon="apple" href="/sdks/ios">
    Native Swift SDK for iOS apps. Supports iOS 13.0+ with Swift 5.7+.
  </Card>
  <Card title="Android SDK" icon="android" href="/sdks/android">
    Native Kotlin SDK for Android apps. Supports API 21+ (Android 5.0).
  </Card>
</Cards>

## Feature Comparison

| Feature | Flutter | iOS | Android |
|---------|---------|-----|---------|
| Deep Link Handling | Yes | Yes | Yes |
| Deferred Deep Linking | Yes | Yes | Yes |
| Deterministic Matching | Yes (Android) | No | Yes (Install Referrer) |
| Probabilistic Matching | Yes | Yes | Yes |
| Event Tracking | Yes | Yes | Yes |
| Revenue Tracking | Yes | Yes | Yes |
| Create Links (SDK) | Yes | Yes | Yes |
| Async/Await | Yes | Yes | Yes |
| Callback API | Yes | Yes | Yes |

## Quick Comparison

### Installation

<Tabs>
<Tab name="Flutter">

```yaml
dependencies:
  clippr: ^0.0.4
```

</Tab>
<Tab name="iOS">

```swift
// Swift Package Manager
.package(url: "https://github.com/nexlabstudio/clippr-ios.git", from: "0.0.4")
```

</Tab>
<Tab name="Android">

```kotlin
implementation("xyz.useclippr:clippr:0.0.4")
```

</Tab>
</Tabs>

### Initialization

<Tabs>
<Tab name="Flutter">

```dart
await Clippr.initialize(apiKey: 'YOUR_API_KEY');
```

</Tab>
<Tab name="iOS">

```swift
Clippr.initialize(apiKey: "YOUR_API_KEY")
```

</Tab>
<Tab name="Android">

```kotlin
Clippr.initialize(context = this, apiKey = "YOUR_API_KEY")
```

</Tab>
</Tabs>

### Get Initial Link

<Tabs>
<Tab name="Flutter">

```dart
final link = await Clippr.getInitialLink();
```

</Tab>
<Tab name="iOS">

```swift
let link = await Clippr.getInitialLink()
```

</Tab>
<Tab name="Android">

```kotlin
val link = Clippr.getInitialLink() // suspend function
```

</Tab>
</Tabs>

### Link Callback

<Tabs>
<Tab name="Flutter">

```dart
Clippr.onLink = (link) {
  print(link.path);
};
```

</Tab>
<Tab name="iOS">

```swift
Clippr.onLink = { link in
  print(link.path)
}
```

</Tab>
<Tab name="Android">

```kotlin
Clippr.onLink = { link ->
  println(link.path)
}
```

</Tab>
</Tabs>

## SDK Architecture

All Clippr SDKs follow the same architecture:

```
┌─────────────────────────────────────────┐
│             Your App Code               │
├─────────────────────────────────────────┤
│           Clippr SDK                    │
│  ┌───────────────────────────────────┐  │
│  │  Link Handler                     │  │
│  │  - Universal Links (iOS)          │  │
│  │  - App Links (Android)            │  │
│  │  - Deferred Link Matching         │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │  Event Tracker                    │  │
│  │  - Custom events                  │  │
│  │  - Revenue tracking               │  │
│  └───────────────────────────────────┘  │
│  ┌───────────────────────────────────┐  │
│  │  Link Creator                     │  │
│  │  - Generate short links           │  │
│  │  - Custom aliases                 │  │
│  └───────────────────────────────────┘  │
├─────────────────────────────────────────┤
│           Clippr API                    │
│       https://api.clppr.xyz             │
└─────────────────────────────────────────┘
```

## Data Models

All SDKs share the same data models:

### ClipprLink

| Property | Type | Description |
|----------|------|-------------|
| `path` | String | The deep link path (e.g., `/product/123`) |
| `metadata` | Map/Dictionary | Custom metadata attached to the link |
| `attribution` | Attribution | Campaign attribution data |
| `matchType` | MatchType | How the link was matched |
| `confidence` | Double | Match confidence score (0.0 - 1.0) |

### Attribution

| Property | Type | Description |
|----------|------|-------------|
| `campaign` | String? | Campaign name |
| `source` | String? | Traffic source |
| `medium` | String? | Marketing medium |

### MatchType

| Value | Description |
|-------|-------------|
| `direct` | User clicked link with app installed |
| `deterministic` | Matched via Install Referrer (Android) |
| `probabilistic` | Matched via device fingerprinting |
| `none` | No match found (organic install) |

### LinkParameters

| Property | Type | Description |
|----------|------|-------------|
| `path` | String | Deep link path |
| `metadata` | Map/Dictionary | Custom metadata |
| `campaign` | String? | Campaign name |
| `source` | String? | Traffic source |
| `medium` | String? | Marketing medium |
| `alias` | String? | Custom short code |
| `socialTags` | SocialMetaTags? | Open Graph tags |

## Requirements

| SDK | Minimum Version |
|-----|-----------------|
| Flutter | Flutter 3.10+, Dart 3.10+ |
| iOS | iOS 13.0+, Swift 5.7+, Xcode 14+ |
| Android | API 21+ (Android 5.0), Kotlin 1.9+ |

## Next Steps

Choose your platform to get started:

- [Flutter SDK](/sdks/flutter) - Full Flutter integration guide
- [iOS SDK](/sdks/ios) - Full iOS integration guide
- [Android SDK](/sdks/android) - Full Android integration guide

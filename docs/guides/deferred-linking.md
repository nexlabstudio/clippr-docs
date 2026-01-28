---
title: Deferred Deep Linking
description: How deferred deep linking works in Clippr
---

# Deferred Deep Linking

Deferred deep linking allows you to attribute users and deliver them to specific content even when they didn't have your app installed when they clicked the link.

## The Problem

Without deferred deep linking:

```
1. User sees ad for shoes on Instagram
2. Clicks link → App not installed
3. Redirected to App Store
4. Installs app
5. Opens app → Lands on home screen 😞
   (Lost: which shoes they wanted, that they came from Instagram)
```

## The Solution

With Clippr deferred deep linking:

```
1. User sees ad for shoes on Instagram
2. Clicks link → Clippr records click + device fingerprint
3. Redirected to App Store
4. Installs app
5. Opens app → SDK matches device → Returns link data
6. App navigates to shoes page 🎉
   (Knows: shoes ID, came from Instagram, part of summer campaign)
```

## How Matching Works

Clippr uses multiple strategies to match clicks to installs:

### 1. Install Referrer (Android Only)

**Accuracy: 100%**

When a user installs from Play Store after clicking a link, Google passes referrer data directly to the app. This is deterministic and cannot fail.

```
Click → Play Store installs with referrer → App reads referrer → Perfect match
```

### 2. Deterministic Matching

**Accuracy: 100%**

If the user has an advertising identifier (IDFA on iOS, GAID on Android) and consented to tracking:

```
Click captures IDFA/GAID → Install sends same ID → Exact match
```

<Info>
On iOS 14.5+, this requires App Tracking Transparency (ATT) permission.
</Info>

### 3. Probabilistic Matching

**Accuracy: ~85-95%**

When deterministic methods aren't available, Clippr uses device fingerprinting:

**Signals collected at click:**
- IP address
- User agent
- Device model
- Screen resolution
- Timezone
- Language
- OS version

**Matching algorithm:**
- Finds recent clicks with similar fingerprints
- Calculates confidence score
- Returns match if confidence exceeds threshold

## Match Types in Code

```dart
final link = await Clippr.getInitialLink();
if (link != null) {
  switch (link.matchType) {
    case MatchType.direct:
      // User clicked with app installed
      // 100% confidence
      break;

    case MatchType.deterministic:
      // Matched via Install Referrer or IDFA/GAID
      // 100% confidence
      break;

    case MatchType.probabilistic:
      // Matched via fingerprinting
      // Check link.confidence (0.0 - 1.0)
      if (link.confidence! > 0.8) {
        // High confidence match
      }
      break;

    case MatchType.none:
      // Organic install, no link clicked
      break;
  }
}
```

## Attribution Window

The time between click and install matters:

| Window | Match Rate | Use Case |
|--------|------------|----------|
| < 1 hour | 95%+ | Same session installs |
| 1-24 hours | 85-95% | Next day installs |
| 24-72 hours | 70-85% | Delayed installs |
| > 72 hours | < 70% | Long consideration periods |

Clippr's default window is 72 hours. Beyond this, probabilistic matching becomes less reliable.

## Maximizing Match Rate

### 1. Prompt for Tracking Permission (iOS)

Request ATT permission for higher accuracy:

```swift
import AppTrackingTransparency

ATTrackingManager.requestTrackingAuthorization { status in
    // Initialize Clippr after permission is determined
    Clippr.initialize(apiKey: "YOUR_API_KEY")
}
```

### 2. Initialize Early

Initialize the SDK as early as possible:

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Clippr.initialize(apiKey: 'YOUR_API_KEY');
  runApp(MyApp());
}
```

### 3. Call getInitialLink() Once

The SDK performs the match on first call and caches the result:

```dart
// Do this once, early in app lifecycle
final link = await Clippr.getInitialLink();
```

### 4. Use Install Referrer (Android)

The SDK automatically uses Install Referrer when available. Ensure your Play Store listing is properly configured.

## One-Time Matching

Deferred matching is one-time by design:

1. User clicks link
2. User installs app
3. First app open → SDK matches → Returns link
4. Subsequent opens → Returns `null`

This prevents:
- Counting the same install multiple times
- Showing the same deep link content repeatedly
- Attribution fraud

## Handling Low Confidence Matches

For probabilistic matches with low confidence:

```dart
void handleDeepLink(ClipprLink link) {
  if (link.matchType == MatchType.probabilistic) {
    final confidence = link.confidence ?? 0;

    if (confidence < 0.5) {
      // Low confidence - might be wrong user
      // Options:
      // 1. Ignore the link
      // 2. Show confirmation: "Were you looking for [X]?"
      // 3. Log for analysis but don't navigate
      return;
    }
  }

  // High confidence - proceed normally
  navigateTo(link.path);
}
```

## Debugging Deferred Links

Enable debug mode to see matching details:

```dart
await Clippr.initialize(apiKey: 'YOUR_API_KEY', debug: true);
```

Logs will show:
- Click lookup results
- Fingerprint comparison
- Confidence calculation
- Final match decision

## Common Issues

### Match Rate Lower Than Expected

1. **Long install delay**: Users installing days after clicking
2. **VPN/proxy usage**: Changes IP between click and install
3. **Shared devices**: Multiple users on same device
4. **Privacy settings**: Limited tracking enabled

### No Match Found

1. **User never clicked a link**: Organic install
2. **Click expired**: Beyond attribution window
3. **Different device**: Clicked on one device, installed on another
4. **SDK not initialized**: Check initialization order

## Next Steps

- [Attribution Guide](/guides/attribution) - Understanding attribution data
- [SDK Quick Start](/getting-started/quickstart) - Implement deferred linking

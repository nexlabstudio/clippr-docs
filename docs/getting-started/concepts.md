---
title: Core Concepts
description: Understand deep linking, deferred deep linking, and attribution
---

# Core Concepts

This guide explains the key concepts behind Clippr and mobile deep linking.

## What is Deep Linking?

Deep linking allows you to link directly to specific content within your mobile app, rather than just opening the app's home screen.

```
Regular Link:
https://yourwebsite.com/products/shoes
→ Opens website in browser

Deep Link:
https://yourapp.clppr.xyz/products/shoes
→ Opens your app directly to the shoes product page
```

### Types of Deep Links

| Type | Description | Example |
|------|-------------|---------|
| **Traditional Deep Link** | Custom URL scheme | `myapp://product/123` |
| **Universal Link (iOS)** | HTTPS link that opens app | `https://yourapp.clppr.xyz/product/123` |
| **App Link (Android)** | HTTPS link that opens app | `https://yourapp.clppr.xyz/product/123` |

Clippr uses Universal Links and App Links because they:
- Work seamlessly (no "Open in App?" prompts)
- Fall back to web gracefully
- Are more secure (domain verification required)

## Deferred Deep Linking

Deferred deep linking solves a critical problem: **What happens when the user doesn't have your app installed?**

### The Problem

```
Without deferred deep linking:
1. User clicks https://yourapp.clppr.xyz/promo/summer
2. User doesn't have app → Redirected to App Store
3. User installs app
4. User opens app → Lands on home screen 😞
   (The original link context is lost!)
```

### The Solution

```
With Clippr's deferred deep linking:
1. User clicks https://yourapp.clppr.xyz/promo/summer
2. Clippr records click + device fingerprint
3. User doesn't have app → Redirected to App Store
4. User installs and opens app
5. SDK calls Clippr API → Matches fingerprint → Returns link data
6. App navigates to /promo/summer 🎉
```

### How Matching Works

Clippr uses multiple strategies to match clicks to installs:

| Method | Platform | Accuracy | How It Works |
|--------|----------|----------|--------------|
| **Install Referrer** | Android | 100% | Google Play passes referrer data directly |
| **Deterministic** | Both | 100% | Advertising ID (IDFA/GAID) matching |
| **Probabilistic** | Both | ~95% | Device fingerprinting (IP, device model, etc.) |

The SDK automatically uses the most accurate method available.

## Attribution

Attribution answers the question: **Where did this user come from?**

### UTM-Style Parameters

When creating links, you can add attribution parameters:

| Parameter | Description | Example |
|-----------|-------------|---------|
| **campaign** | Marketing campaign name | `summer_sale_2024` |
| **source** | Traffic source | `facebook`, `google`, `email` |
| **medium** | Marketing medium | `social`, `cpc`, `newsletter` |

### Example

```
Link: https://yourapp.clppr.xyz/promo?campaign=summer&source=instagram&medium=story

When user installs and opens:
{
  "path": "/promo",
  "attribution": {
    "campaign": "summer",
    "source": "instagram",
    "medium": "story"
  }
}
```

### Tracking in Dashboard

The Clippr dashboard shows:
- Clicks per campaign/source/medium
- Installs attributed to each link
- Match rate (% of clicks that led to matched installs)
- Top performing links and campaigns

## Match Types

When the SDK retrieves a deep link, it includes a `matchType` indicating how the match was made:

| Match Type | Description | Confidence |
|------------|-------------|------------|
| `direct` | User clicked link with app already installed | 100% |
| `deterministic` | Matched via Install Referrer or Advertising ID | 100% |
| `probabilistic` | Matched via device fingerprinting | 0-100% (varies) |
| `none` | No match found (organic install) | N/A |

### Using Match Type in Code

```dart
final link = await Clippr.getInitialLink();
if (link != null) {
  switch (link.matchType) {
    case MatchType.direct:
      // User clicked link with app installed
      break;
    case MatchType.deterministic:
      // 100% confident this is the right user
      break;
    case MatchType.probabilistic:
      // Check confidence score
      if (link.confidence != null && link.confidence! > 0.8) {
        // High confidence match
      }
      break;
    case MatchType.none:
      // Organic install, no attribution
      break;
  }
}
```

## Link Lifecycle

Understanding the complete link journey:

```
┌─────────────────────────────────────────────────────────────┐
│                      LINK CREATION                          │
├─────────────────────────────────────────────────────────────┤
│ Dashboard/API/SDK creates link with:                        │
│ - Deep link path (/product/123)                            │
│ - Attribution (campaign, source, medium)                   │
│ - Social preview (title, description, image)               │
│ - Fallback URLs (iOS, Android, Web)                        │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                       LINK CLICK                            │
├─────────────────────────────────────────────────────────────┤
│ Clippr edge server:                                         │
│ 1. Records click with device fingerprint                   │
│ 2. Checks if Universal Link / App Link                     │
│ 3. If app installed → Opens app directly                   │
│ 4. If not installed → Redirects to store/fallback          │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                       APP OPENS                             │
├─────────────────────────────────────────────────────────────┤
│ SDK initialization:                                         │
│ 1. Checks for direct Universal/App Link                    │
│ 2. If no direct link, calls match API                      │
│ 3. API matches device fingerprint to recent click          │
│ 4. Returns link data with attribution                      │
│ 5. App navigates to deep link path                         │
└─────────────────────────────────────────────────────────────┘
```

## Universal Links (iOS) & App Links (Android)

These are the technologies that enable seamless deep linking.

### How They Work

1. **Domain Verification**: Apple/Google verify that you own the domain
2. **Configuration Files**:
   - iOS: `/.well-known/apple-app-site-association` (AASA)
   - Android: `/.well-known/assetlinks.json`
3. **Link Handling**: When user clicks a verified link, the OS opens your app directly

### Clippr Handles This For You

- Clippr automatically hosts AASA and assetlinks.json files
- Files are served from your app's subdomain (e.g., `yourapp.clppr.xyz`)
- You just need to configure your app's bundle ID/package name in the dashboard

## Event Tracking

Beyond link attribution, Clippr can track in-app events:

### Standard Events

```dart
// Track a simple event
await Clippr.track('signup_completed');

// Track with parameters
await Clippr.track('add_to_cart', params: {
  'product_id': '123',
  'price': 29.99,
});

// Track revenue
await Clippr.trackRevenue(
  'purchase',
  revenue: 99.99,
  currency: 'USD',
);
```

### Why Track Events?

- Measure conversion from link click → desired action
- Calculate ROI per campaign/source
- Identify high-value acquisition channels

## Next Steps

Now that you understand the concepts:

- [Quick Start](/getting-started/quickstart) - Get integrated in 5 minutes
- [Dashboard Setup](/getting-started/dashboard-setup) - Configure your apps
- [Attribution Guide](/guides/attribution) - Deep dive into attribution
- [Deferred Linking Guide](/guides/deferred-linking) - Advanced deferred linking

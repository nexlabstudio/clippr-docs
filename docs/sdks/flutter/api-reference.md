---
title: API Reference
description: Complete API reference for the Clippr Flutter SDK
---

# API Reference

Complete reference for the Clippr Flutter SDK.

## Clippr Class

The main SDK class. All methods are static.

### initialize

Initialize the Clippr SDK. Must be called before any other methods.

```dart
static Future<void> initialize({
  required String apiKey,
  bool debug = false,
})
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `apiKey` | `String` | Yes | Your app's API key from the dashboard |
| `debug` | `bool` | No | Enable debug logging (default: `false`) |

**Example:**

```dart
await Clippr.initialize(
  apiKey: 'YOUR_API_KEY',
  debug: true,
);
```

---

### getInitialLink

Get the deep link that opened the app (direct or deferred).

```dart
static Future<ClipprLink?> getInitialLink()
```

**Returns:** `Future<ClipprLink?>` - The link data, or `null` if app wasn't opened via a link.

**Example:**

```dart
final link = await Clippr.getInitialLink();
if (link != null) {
  print('App opened with path: ${link.path}');
}
```

<Info>
This method should only be called once per app launch. For deferred links, the result is cached and cleared after the first call.
</Info>

---

### onLink

Callback for links received while the app is running.

```dart
static void Function(ClipprLink link)? onLink
```

**Example:**

```dart
Clippr.onLink = (link) {
  print('Received link: ${link.path}');
};

// Clear the listener
Clippr.onLink = null;
```

---

### track

Track a custom event.

```dart
static Future<void> track(
  String eventName, {
  Map<String, dynamic>? params,
})
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `eventName` | `String` | Yes | Name of the event |
| `params` | `Map<String, dynamic>?` | No | Additional event parameters |

**Example:**

```dart
await Clippr.track('add_to_cart', params: {
  'product_id': '12345',
  'price': 29.99,
});
```

---

### trackRevenue

Track a revenue event.

```dart
static Future<void> trackRevenue(
  String eventName, {
  required double revenue,
  required String currency,
  Map<String, dynamic>? params,
})
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `eventName` | `String` | Yes | Name of the event |
| `revenue` | `double` | Yes | Revenue amount |
| `currency` | `String` | Yes | ISO 4217 currency code (e.g., `USD`) |
| `params` | `Map<String, dynamic>?` | No | Additional event parameters |

**Example:**

```dart
await Clippr.trackRevenue(
  'purchase',
  revenue: 99.99,
  currency: 'USD',
  params: {'order_id': 'ORDER-123'},
);
```

---

### createLink

Create a short link programmatically.

```dart
static Future<ShortLink> createLink(LinkParameters parameters)
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `parameters` | `LinkParameters` | Yes | Link configuration |

**Returns:** `Future<ShortLink>` - The created short link.

**Throws:** `ClipprException` on error (e.g., alias already taken).

**Example:**

```dart
final params = LinkParameters(
  path: '/product/123',
  campaign: 'summer_sale',
);

final shortLink = await Clippr.createLink(params);
print('URL: ${shortLink.url}');
```

---

### handle

Manually handle a URL.

```dart
static Future<bool> handle(Uri uri)
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `uri` | `Uri` | Yes | The URL to handle |

**Returns:** `Future<bool>` - `true` if the URL was a Clippr link and was handled.

**Example:**

```dart
final uri = Uri.parse('https://yourapp.clppr.xyz/promo/summer');
final handled = await Clippr.handle(uri);
```

---

## Data Classes

### ClipprLink

Represents a deep link with its associated data.

```dart
class ClipprLink {
  final String path;
  final Map<String, dynamic>? metadata;
  final Attribution? attribution;
  final MatchType matchType;
  final double? confidence;
}
```

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `path` | `String` | The deep link path (e.g., `/product/123`) |
| `metadata` | `Map<String, dynamic>?` | Custom metadata attached to the link |
| `attribution` | `Attribution?` | Campaign attribution data |
| `matchType` | `MatchType` | How the link was matched |
| `confidence` | `double?` | Match confidence (0.0 - 1.0) for probabilistic matches |

---

### Attribution

Campaign attribution data.

```dart
class Attribution {
  final String? campaign;
  final String? source;
  final String? medium;
}
```

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `campaign` | `String?` | Campaign name (e.g., `summer_sale`) |
| `source` | `String?` | Traffic source (e.g., `facebook`, `email`) |
| `medium` | `String?` | Marketing medium (e.g., `social`, `cpc`) |

---

### MatchType

Enum indicating how a link was matched.

```dart
enum MatchType {
  direct,
  deterministic,
  probabilistic,
  none,
}
```

| Value | Description |
|-------|-------------|
| `direct` | User clicked link with app already installed |
| `deterministic` | Matched via Install Referrer (Android, 100% accurate) |
| `probabilistic` | Matched via device fingerprinting |
| `none` | No match found |

---

### LinkParameters

Parameters for creating a short link.

```dart
class LinkParameters {
  final String path;
  final Map<String, dynamic>? metadata;
  final String? campaign;
  final String? source;
  final String? medium;
  final String? alias;
  final SocialMetaTags? socialTags;
}
```

**Constructor:**

```dart
LinkParameters({
  required String path,
  Map<String, dynamic>? metadata,
  String? campaign,
  String? source,
  String? medium,
  String? alias,
  SocialMetaTags? socialTags,
})
```

**Properties:**

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `path` | `String` | Yes | Deep link path |
| `metadata` | `Map<String, dynamic>?` | No | Custom metadata |
| `campaign` | `String?` | No | Campaign name |
| `source` | `String?` | No | Traffic source |
| `medium` | `String?` | No | Marketing medium |
| `alias` | `String?` | No | Custom short code |
| `socialTags` | `SocialMetaTags?` | No | Social preview tags |

---

### SocialMetaTags

Open Graph tags for social media previews.

```dart
class SocialMetaTags {
  final String? title;
  final String? description;
  final String? imageUrl;
}
```

**Constructor:**

```dart
SocialMetaTags({
  String? title,
  String? description,
  String? imageUrl,
})
```

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `title` | `String?` | Title for social previews (og:title) |
| `description` | `String?` | Description for social previews (og:description) |
| `imageUrl` | `String?` | Image URL for social previews (og:image) |

---

### ShortLink

A created short link.

```dart
class ShortLink {
  final String url;
  final String shortCode;
  final String path;
}
```

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `url` | `String` | The full short URL |
| `shortCode` | `String` | The short code or alias |
| `path` | `String` | The original deep link path |

---

## Exceptions

### ClipprException

Base exception for Clippr errors.

```dart
class ClipprException implements Exception {
  final String message;
  final String? code;
}
```

**Properties:**

| Property | Type | Description |
|----------|------|-------------|
| `message` | `String` | Error message |
| `code` | `String?` | Error code (e.g., `ALIAS_TAKEN`) |

**Common Error Codes:**

| Code | Description |
|------|-------------|
| `ALIAS_TAKEN` | The requested alias is already in use |
| `INVALID_API_KEY` | The API key is invalid |
| `NETWORK_ERROR` | Network request failed |
| `RATE_LIMITED` | Too many requests |

---

## Platform Specifics

### iOS

The SDK automatically handles:
- Universal Link resolution
- App Tracking Transparency (ATT) for IDFA access
- Deferred deep link matching

### Android

The SDK automatically handles:
- App Link resolution
- Install Referrer for deterministic matching
- Google Advertising ID (if available)

---

## Thread Safety

All Clippr methods are safe to call from any thread. The SDK handles thread synchronization internally.

---

## Memory Management

The SDK uses minimal memory and cleans up resources appropriately. The `onLink` callback is weakly referenced to prevent memory leaks.

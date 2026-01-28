---
title: Creating Links
description: Create short links programmatically in your Flutter app
---

# Creating Links

Create Clippr short links programmatically from your Flutter app. This is useful for sharing features, referral programs, and dynamic content sharing.

## Basic Link Creation

```dart
import 'package:clippr/clippr.dart';

Future<void> createLink() async {
  final params = LinkParameters(
    path: '/product/123',
  );

  try {
    final shortLink = await Clippr.createLink(params);
    print('Short URL: ${shortLink.url}');
    // Output: https://yourapp.clppr.xyz/abc123
  } catch (e) {
    print('Error creating link: $e');
  }
}
```

## Link Parameters

### Full Parameter Example

```dart
final params = LinkParameters(
  // Required: The deep link path
  path: '/product/123',

  // Optional: Custom metadata (accessible in your app)
  metadata: {
    'product_name': 'Cool Shoes',
    'price': 99.99,
    'color': 'blue',
  },

  // Optional: Attribution parameters
  campaign: 'summer_sale',
  source: 'app_share',
  medium: 'social',

  // Optional: Custom short code (alias)
  alias: 'summer-shoes',

  // Optional: Social preview tags
  socialTags: SocialMetaTags(
    title: 'Check out these shoes!',
    description: '50% off summer sale - Limited time only',
    imageUrl: 'https://example.com/shoes.jpg',
  ),
);

final shortLink = await Clippr.createLink(params);
// Result: https://yourapp.clppr.xyz/summer-shoes
```

### Parameter Reference

| Parameter | Type | Description |
|-----------|------|-------------|
| `path` | `String` | **Required**. Deep link path (e.g., `/product/123`) |
| `metadata` | `Map<String, dynamic>?` | Custom data accessible when link is opened |
| `campaign` | `String?` | Campaign name for attribution |
| `source` | `String?` | Traffic source (e.g., `app_share`, `referral`) |
| `medium` | `String?` | Marketing medium (e.g., `social`, `email`) |
| `alias` | `String?` | Custom short code (must be unique) |
| `socialTags` | `SocialMetaTags?` | Open Graph tags for social previews |

### Social Meta Tags

Control how your link appears when shared on social media:

```dart
final socialTags = SocialMetaTags(
  title: 'Product Name',           // og:title
  description: 'Product desc...',  // og:description
  imageUrl: 'https://...',         // og:image
);
```

## Common Use Cases

### Product Sharing

```dart
Future<String> shareProduct(Product product) async {
  final params = LinkParameters(
    path: '/product/${product.id}',
    metadata: {
      'name': product.name,
      'price': product.price,
    },
    source: 'product_share',
    socialTags: SocialMetaTags(
      title: product.name,
      description: product.description,
      imageUrl: product.imageUrl,
    ),
  );

  final shortLink = await Clippr.createLink(params);
  return shortLink.url;
}

// Usage with share sheet
void onShareTap(Product product) async {
  final url = await shareProduct(product);
  Share.share('Check out ${product.name}! $url');
}
```

### Referral Links

```dart
Future<String> createReferralLink(String userId) async {
  final params = LinkParameters(
    path: '/referral',
    metadata: {
      'referrer_id': userId,
      'reward': '10%_discount',
    },
    campaign: 'referral_program',
    source: 'user_invite',
    medium: 'referral',
    alias: 'ref-$userId', // Each user gets unique alias
    socialTags: SocialMetaTags(
      title: 'Join MyApp and get 10% off!',
      description: 'Your friend invited you to join MyApp.',
      imageUrl: 'https://example.com/referral-banner.jpg',
    ),
  );

  final shortLink = await Clippr.createLink(params);
  return shortLink.url;
}
```

### Content Sharing

```dart
Future<String> shareArticle(Article article) async {
  final params = LinkParameters(
    path: '/article/${article.slug}',
    campaign: 'content_share',
    source: 'in_app',
    socialTags: SocialMetaTags(
      title: article.title,
      description: article.excerpt,
      imageUrl: article.featuredImage,
    ),
  );

  final shortLink = await Clippr.createLink(params);
  return shortLink.url;
}
```

### Event Invitations

```dart
Future<String> createEventInvite(Event event, String hostId) async {
  final params = LinkParameters(
    path: '/event/${event.id}/join',
    metadata: {
      'event_name': event.name,
      'host_id': hostId,
      'date': event.date.toIso8601String(),
    },
    campaign: 'event_invite',
    source: 'host_share',
    socialTags: SocialMetaTags(
      title: 'You\'re invited to ${event.name}',
      description: 'Join us on ${event.formattedDate}',
      imageUrl: event.coverImage,
    ),
  );

  final shortLink = await Clippr.createLink(params);
  return shortLink.url;
}
```

## ShortLink Response

The `createLink` method returns a `ShortLink` object:

```dart
final shortLink = await Clippr.createLink(params);

// The full short URL
print(shortLink.url);
// Output: https://yourapp.clppr.xyz/abc123

// The short code (or alias if provided)
print(shortLink.shortCode);
// Output: abc123 (or your custom alias)

// The original deep link path
print(shortLink.path);
// Output: /product/123
```

## Error Handling

```dart
Future<String?> createLinkSafely(LinkParameters params) async {
  try {
    final shortLink = await Clippr.createLink(params);
    return shortLink.url;
  } on ClipprException catch (e) {
    // Handle Clippr-specific errors
    print('Clippr error: ${e.message}');

    if (e.code == 'ALIAS_TAKEN') {
      // The custom alias is already in use
      // Retry without alias or use a different one
    }

    return null;
  } catch (e) {
    // Handle network or other errors
    print('Error creating link: $e');
    return null;
  }
}
```

## Best Practices

### 1. Cache Referral Links

Don't create a new link every time - cache per-user referral links:

```dart
class ReferralService {
  final Map<String, String> _cache = {};

  Future<String> getReferralLink(String userId) async {
    if (_cache.containsKey(userId)) {
      return _cache[userId]!;
    }

    final url = await _createReferralLink(userId);
    _cache[userId] = url;
    return url;
  }
}
```

### 2. Use Meaningful Aliases

```dart
// Good - descriptive and memorable
alias: 'summer-sale-2024'
alias: 'ref-${username}'
alias: '${productSlug}-share'

// Avoid - not memorable
alias: 'a1b2c3'
```

### 3. Include Attribution

Always include attribution parameters to track where shares convert:

```dart
final params = LinkParameters(
  path: '/product/123',
  campaign: 'user_share',    // What campaign
  source: 'share_button',    // Where in app
  medium: 'social',          // Share type
);
```

### 4. Optimize Social Tags

Test how your links appear on different platforms:

```dart
final socialTags = SocialMetaTags(
  title: 'Keep under 60 chars for best display',
  description: 'Keep under 155 chars. Be compelling!',
  imageUrl: 'https://...', // Use 1200x630px for best results
);
```

## Next Steps

- [Event Tracking](/sdks/flutter/event-tracking) - Track conversions from shared links
- [API Reference](/sdks/flutter/api-reference) - Complete API documentation
- [Attribution Guide](/guides/attribution) - Understanding attribution

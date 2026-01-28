---
title: Event Tracking
description: Track events and revenue in your Flutter app
---

# Event Tracking

Track in-app events and revenue to measure the effectiveness of your deep links and campaigns.

## Why Track Events?

Event tracking helps you:
- Measure conversion from link click → desired action
- Calculate ROI per campaign/source
- Identify high-value acquisition channels
- Understand user behavior after deep link attribution

## Track Simple Events

Track when users complete key actions:

```dart
// Track a simple event
await Clippr.track('signup_completed');

// Track with event name only
await Clippr.track('onboarding_started');
await Clippr.track('tutorial_finished');
await Clippr.track('profile_created');
```

## Track Events with Parameters

Add context to your events:

```dart
// Track add to cart with product details
await Clippr.track('add_to_cart', params: {
  'product_id': '12345',
  'product_name': 'Premium Headphones',
  'price': 149.99,
  'quantity': 1,
  'category': 'electronics',
});

// Track search
await Clippr.track('search', params: {
  'query': 'wireless headphones',
  'results_count': 24,
});

// Track content view
await Clippr.track('view_content', params: {
  'content_id': 'article-456',
  'content_type': 'blog_post',
  'author': 'john_doe',
});
```

## Track Revenue

Track purchases and revenue for ROI analysis:

```dart
// Track a purchase
await Clippr.trackRevenue(
  'purchase',
  revenue: 99.99,
  currency: 'USD',
  params: {
    'order_id': 'ORDER-12345',
    'product_ids': ['prod-1', 'prod-2'],
    'coupon_code': 'SUMMER20',
  },
);

// Track subscription
await Clippr.trackRevenue(
  'subscription_started',
  revenue: 9.99,
  currency: 'USD',
  params: {
    'plan': 'premium_monthly',
    'trial': false,
  },
);

// Track in-app purchase
await Clippr.trackRevenue(
  'in_app_purchase',
  revenue: 4.99,
  currency: 'USD',
  params: {
    'item': 'coin_pack_500',
    'store': 'app_store',
  },
);
```

### Currency Codes

Use standard ISO 4217 currency codes:

| Currency | Code |
|----------|------|
| US Dollar | `USD` |
| Euro | `EUR` |
| British Pound | `GBP` |
| Japanese Yen | `JPY` |
| Nigerian Naira | `NGN` |

## Common Event Examples

### E-commerce Events

```dart
// Product viewed
await Clippr.track('view_product', params: {
  'product_id': 'SKU-123',
  'product_name': 'Running Shoes',
  'price': 89.99,
  'category': 'footwear',
});

// Add to cart
await Clippr.track('add_to_cart', params: {
  'product_id': 'SKU-123',
  'quantity': 1,
  'price': 89.99,
});

// Begin checkout
await Clippr.track('begin_checkout', params: {
  'cart_value': 189.98,
  'item_count': 2,
});

// Purchase completed
await Clippr.trackRevenue('purchase', revenue: 189.98, currency: 'USD', params: {
  'order_id': 'ORD-789',
  'shipping_method': 'express',
  'payment_method': 'credit_card',
});
```

### User Lifecycle Events

```dart
// Registration
await Clippr.track('sign_up', params: {
  'method': 'email', // or 'google', 'apple', etc.
});

// Login
await Clippr.track('login', params: {
  'method': 'email',
});

// Profile completed
await Clippr.track('complete_profile', params: {
  'fields_completed': ['name', 'avatar', 'bio'],
});

// Subscription started
await Clippr.trackRevenue('subscribe', revenue: 9.99, currency: 'USD', params: {
  'plan': 'premium',
  'billing_cycle': 'monthly',
});
```

### Content & Engagement Events

```dart
// Article read
await Clippr.track('read_article', params: {
  'article_id': 'ART-456',
  'read_time_seconds': 180,
  'scroll_depth': 0.85,
});

// Video watched
await Clippr.track('watch_video', params: {
  'video_id': 'VID-789',
  'duration_seconds': 300,
  'watched_seconds': 245,
  'completed': false,
});

// Share
await Clippr.track('share', params: {
  'content_type': 'product',
  'content_id': 'PROD-123',
  'share_method': 'whatsapp',
});
```

### Referral Events

```dart
// Referral link shared
await Clippr.track('referral_shared', params: {
  'share_method': 'copy_link',
});

// Referral converted (when invited user signs up)
await Clippr.track('referral_converted', params: {
  'referred_user_id': 'USER-456',
});

// Referral reward earned
await Clippr.track('referral_reward', params: {
  'reward_type': 'credit',
  'reward_value': 10.00,
});
```

## Best Practices

### 1. Track After Successful Completion

Only track events after the action is truly complete:

```dart
// Good - track after API confirms success
Future<void> completePurchase() async {
  try {
    final order = await _orderService.createOrder(cart);

    // Track only after successful order creation
    await Clippr.trackRevenue(
      'purchase',
      revenue: order.total,
      currency: 'USD',
      params: {'order_id': order.id},
    );
  } catch (e) {
    // Don't track failed purchases
    print('Order failed: $e');
  }
}
```

### 2. Use Consistent Event Names

Establish naming conventions:

```dart
// Good - consistent snake_case naming
'sign_up'
'add_to_cart'
'begin_checkout'
'purchase_completed'

// Avoid - inconsistent naming
'SignUp'
'addToCart'
'checkout-started'
'PURCHASE'
```

### 3. Don't Over-Track

Track meaningful events, not every interaction:

```dart
// Good - meaningful conversion events
await Clippr.track('sign_up');
await Clippr.track('purchase');
await Clippr.track('subscription_started');

// Avoid - too granular
await Clippr.track('button_tapped');
await Clippr.track('screen_scrolled');
await Clippr.track('keyboard_shown');
```

### 4. Include Relevant Context

Add parameters that help with analysis:

```dart
// Good - actionable context
await Clippr.track('add_to_cart', params: {
  'product_id': product.id,
  'category': product.category,
  'price': product.price,
  'source': 'product_page', // Where the action happened
});

// Less useful - missing context
await Clippr.track('add_to_cart', params: {
  'product_id': product.id,
});
```

## Error Handling

Event tracking is fire-and-forget by default, but you can handle errors:

```dart
Future<void> trackSafely(String event, {Map<String, dynamic>? params}) async {
  try {
    await Clippr.track(event, params: params);
  } catch (e) {
    // Log but don't block user flow
    print('Failed to track $event: $e');
    // Optionally queue for retry
    _eventQueue.add(EventData(event, params));
  }
}
```

## Viewing Analytics

Track events appear in the Clippr dashboard:

1. Go to **Analytics** in the dashboard
2. View events by campaign, source, and medium
3. See conversion rates from click → event
4. Analyze revenue attribution

## Next Steps

- [API Reference](/sdks/flutter/api-reference) - Complete API documentation
- [Attribution Guide](/guides/attribution) - Deep dive into attribution
- [Dashboard Analytics](/dashboard/analytics) - Using the analytics dashboard

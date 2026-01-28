---
title: Event Tracking
description: Track events and revenue in your iOS app
---

# Event Tracking

Track in-app events and revenue to measure the effectiveness of your deep links and campaigns.

## Track Simple Events

```swift
// Using async/await
try await Clippr.track("signup_completed")

// Using completion handler
Clippr.track("signup_completed") { error in
    if let error = error {
        print("Failed to track: \(error)")
    }
}
```

## Track Events with Parameters

```swift
try await Clippr.track("add_to_cart", params: [
    "product_id": "12345",
    "product_name": "Premium Headphones",
    "price": 149.99,
    "quantity": 1
])

try await Clippr.track("search", params: [
    "query": "wireless headphones",
    "results_count": 24
])
```

## Track Revenue

```swift
// Track a purchase
try await Clippr.trackRevenue(
    "purchase",
    revenue: 99.99,
    currency: "USD",
    params: [
        "order_id": "ORDER-12345",
        "items_count": 3
    ]
)

// Track subscription
try await Clippr.trackRevenue(
    "subscription_started",
    revenue: 9.99,
    currency: "USD",
    params: [
        "plan": "premium_monthly",
        "trial": false
    ]
)

// Using completion handler
Clippr.trackRevenue(
    "in_app_purchase",
    revenue: 4.99,
    currency: "USD",
    params: ["item": "coin_pack"]
) { error in
    if let error = error {
        print("Failed: \(error)")
    }
}
```

## Common Event Examples

### E-commerce

```swift
// Product viewed
try await Clippr.track("view_product", params: [
    "product_id": product.id,
    "name": product.name,
    "price": product.price,
    "category": product.category
])

// Add to cart
try await Clippr.track("add_to_cart", params: [
    "product_id": product.id,
    "quantity": 1,
    "price": product.price
])

// Purchase
try await Clippr.trackRevenue("purchase", revenue: order.total, currency: "USD", params: [
    "order_id": order.id,
    "items_count": order.items.count
])
```

### User Lifecycle

```swift
// Sign up
try await Clippr.track("sign_up", params: [
    "method": "email" // or "apple", "google"
])

// Complete profile
try await Clippr.track("complete_profile", params: [
    "fields": ["name", "avatar", "bio"]
])

// Subscribe
try await Clippr.trackRevenue("subscribe", revenue: 9.99, currency: "USD", params: [
    "plan": "premium",
    "billing": "monthly"
])
```

### Content Engagement

```swift
// Article read
try await Clippr.track("read_article", params: [
    "article_id": article.id,
    "read_time_seconds": 180,
    "scroll_depth": 0.85
])

// Share
try await Clippr.track("share", params: [
    "content_type": "product",
    "content_id": product.id,
    "method": "messages"
])
```

## Best Practices

### Track After Success

```swift
func completePurchase() async throws {
    // Create order first
    let order = try await orderService.createOrder(cart)

    // Only track after success
    try await Clippr.trackRevenue(
        "purchase",
        revenue: order.total,
        currency: "USD",
        params: ["order_id": order.id]
    )
}
```

### Consistent Naming

```swift
// Good - snake_case
"sign_up"
"add_to_cart"
"purchase_completed"

// Avoid - inconsistent
"SignUp"
"addToCart"
```

### Include Context

```swift
// Good - actionable context
try await Clippr.track("add_to_cart", params: [
    "product_id": product.id,
    "category": product.category,
    "price": product.price,
    "source": "product_page"
])
```

## Error Handling

```swift
func trackSafely(_ event: String, params: [String: Any]? = nil) {
    Task {
        do {
            try await Clippr.track(event, params: params)
        } catch {
            // Log but don't block user flow
            print("Track failed: \(error)")
        }
    }
}
```

## Next Steps

- [API Reference](/sdks/ios/api-reference) - Complete API documentation
- [Attribution Guide](/guides/attribution) - Understanding attribution

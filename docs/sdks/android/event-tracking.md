---
title: Event Tracking
description: Track events and revenue in your Android app
---

# Event Tracking

Track in-app events and revenue to measure campaign effectiveness.

## Track Simple Events

```kotlin
// Using coroutines
lifecycleScope.launch {
    Clippr.track("signup_completed")
}

// Using callback
Clippr.track("signup_completed") { error ->
    if (error != null) {
        Log.e("Clippr", "Failed to track", error)
    }
}
```

## Track Events with Parameters

```kotlin
lifecycleScope.launch {
    Clippr.track("add_to_cart", mapOf(
        "product_id" to "12345",
        "product_name" to "Premium Headphones",
        "price" to 149.99,
        "quantity" to 1
    ))
}
```

## Track Revenue

```kotlin
// Track purchase
lifecycleScope.launch {
    Clippr.trackRevenue(
        eventName = "purchase",
        revenue = 99.99,
        currency = "USD",
        params = mapOf(
            "order_id" to "ORDER-12345",
            "items_count" to 3
        )
    )
}

// Track subscription
lifecycleScope.launch {
    Clippr.trackRevenue(
        eventName = "subscription_started",
        revenue = 9.99,
        currency = "USD",
        params = mapOf(
            "plan" to "premium_monthly"
        )
    )
}

// Using callback
Clippr.trackRevenue("in_app_purchase", 4.99, "USD", mapOf("item" to "coins")) { error ->
    if (error != null) Log.e("Clippr", "Failed", error)
}
```

## Common Events

### E-commerce

```kotlin
// Product viewed
Clippr.track("view_product", mapOf(
    "product_id" to product.id,
    "name" to product.name,
    "price" to product.price,
    "category" to product.category
))

// Add to cart
Clippr.track("add_to_cart", mapOf(
    "product_id" to product.id,
    "quantity" to 1,
    "price" to product.price
))

// Purchase
Clippr.trackRevenue("purchase", order.total, "USD", mapOf(
    "order_id" to order.id,
    "items_count" to order.items.size
))
```

### User Lifecycle

```kotlin
// Sign up
Clippr.track("sign_up", mapOf("method" to "email"))

// Complete profile
Clippr.track("complete_profile", mapOf(
    "fields" to listOf("name", "avatar", "bio")
))

// Subscribe
Clippr.trackRevenue("subscribe", 9.99, "USD", mapOf(
    "plan" to "premium",
    "billing" to "monthly"
))
```

## Java Usage

```java
// Track event
Clippr.track("signup_completed", null, error -> {
    if (error != null) {
        Log.e("Clippr", "Failed", error);
    }
});

// Track with params
Map<String, Object> params = new HashMap<>();
params.put("product_id", "123");
params.put("price", 29.99);
Clippr.track("add_to_cart", params, null);

// Track revenue
Clippr.trackRevenue("purchase", 99.99, "USD", params, null);
```

## Best Practices

### Track After Success

```kotlin
suspend fun completePurchase() {
    val order = orderService.createOrder(cart)

    // Only track after success
    Clippr.trackRevenue("purchase", order.total, "USD", mapOf(
        "order_id" to order.id
    ))
}
```

### Consistent Naming

```kotlin
// Good
"sign_up"
"add_to_cart"
"purchase_completed"

// Avoid
"SignUp"
"addToCart"
```

### Include Context

```kotlin
Clippr.track("add_to_cart", mapOf(
    "product_id" to product.id,
    "category" to product.category,
    "price" to product.price,
    "source" to "product_page"  // Where the action happened
))
```

## Error Handling

```kotlin
fun trackSafely(event: String, params: Map<String, Any>? = null) {
    lifecycleScope.launch {
        try {
            Clippr.track(event, params)
        } catch (e: Exception) {
            Log.e("Clippr", "Track failed: $event", e)
        }
    }
}
```

## Next Steps

- [API Reference](/sdks/android/api-reference) - Complete API documentation
- [Attribution Guide](/guides/attribution) - Understanding attribution

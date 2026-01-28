---
title: Creating Links
description: Create short links programmatically in your Android app
---

# Creating Links

Create Clippr short links programmatically from your Android app.

## Basic Link Creation

```kotlin
// Using coroutines
lifecycleScope.launch {
    try {
        val params = LinkParameters(path = "/product/123")
        val shortLink = Clippr.createLink(params)
        Log.d("Clippr", "URL: ${shortLink.url}")
    } catch (e: Exception) {
        Log.e("Clippr", "Failed to create link", e)
    }
}

// Using callback
Clippr.createLink(LinkParameters(path = "/product/123")) { shortLink, error ->
    if (error != null) {
        Log.e("Clippr", "Failed", error)
    } else {
        Log.d("Clippr", "URL: ${shortLink?.url}")
    }
}
```

## Full Parameter Example

```kotlin
val params = LinkParameters(
    // Required
    path = "/product/123",

    // Optional: Custom metadata
    metadata = mapOf(
        "product_name" to "Premium Headphones",
        "price" to 149.99,
        "color" to "black"
    ),

    // Optional: Attribution
    campaign = "summer_sale",
    source = "app_share",
    medium = "social",

    // Optional: Custom alias
    alias = "summer-headphones",

    // Optional: Social preview
    socialTags = SocialMetaTags(
        title = "Check out these headphones!",
        description = "50% off for a limited time",
        imageUrl = "https://example.com/headphones.jpg"
    )
)

val shortLink = Clippr.createLink(params)
// Result: https://yourapp.clppr.xyz/summer-headphones
```

## Common Use Cases

### Product Sharing

```kotlin
suspend fun shareProduct(product: Product) {
    val params = LinkParameters(
        path = "/product/${product.id}",
        metadata = mapOf(
            "name" to product.name,
            "price" to product.price
        ),
        source = "product_share",
        socialTags = SocialMetaTags(
            title = product.name,
            description = product.description,
            imageUrl = product.imageUrl
        )
    )

    try {
        val shortLink = Clippr.createLink(params)

        val shareIntent = Intent(Intent.ACTION_SEND).apply {
            type = "text/plain"
            putExtra(Intent.EXTRA_TEXT, "Check out ${product.name}! ${shortLink.url}")
        }
        startActivity(Intent.createChooser(shareIntent, "Share via"))
    } catch (e: Exception) {
        Toast.makeText(this, "Failed to create link", Toast.LENGTH_SHORT).show()
    }
}
```

### Referral Program

```kotlin
suspend fun createReferralLink(userId: String): String {
    val params = LinkParameters(
        path = "/referral",
        metadata = mapOf(
            "referrer_id" to userId,
            "reward" to "10% discount"
        ),
        campaign = "referral_program",
        source = "user_invite",
        medium = "referral",
        alias = "ref-$userId",
        socialTags = SocialMetaTags(
            title = "Join MyApp and get 10% off!",
            description = "Your friend invited you",
            imageUrl = "https://example.com/referral.jpg"
        )
    )

    return Clippr.createLink(params).url
}
```

### With Jetpack Compose

```kotlin
@Composable
fun ProductScreen(product: Product) {
    var shareUrl by remember { mutableStateOf<String?>(null) }
    val scope = rememberCoroutineScope()
    val context = LocalContext.current

    Column {
        // Product details...

        Button(onClick = {
            scope.launch {
                try {
                    val params = LinkParameters(
                        path = "/product/${product.id}",
                        socialTags = SocialMetaTags(
                            title = product.name,
                            description = product.description
                        )
                    )
                    val link = Clippr.createLink(params)
                    shareUrl = link.url

                    // Share immediately
                    val intent = Intent(Intent.ACTION_SEND).apply {
                        type = "text/plain"
                        putExtra(Intent.EXTRA_TEXT, link.url)
                    }
                    context.startActivity(Intent.createChooser(intent, "Share"))
                } catch (e: Exception) {
                    // Handle error
                }
            }
        }) {
            Text("Share")
        }
    }
}
```

## ShortLink Response

```kotlin
val shortLink = Clippr.createLink(params)

shortLink.url        // https://yourapp.clppr.xyz/abc123
shortLink.shortCode  // abc123 (or custom alias)
shortLink.path       // /product/123
```

## Error Handling

```kotlin
suspend fun createLinkSafely(params: LinkParameters): String? {
    return try {
        Clippr.createLink(params).url
    } catch (e: ClipprException) {
        when (e) {
            is ClipprException.AliasAlreadyExists -> {
                // Retry without alias
                val newParams = params.copy(alias = null)
                Clippr.createLink(newParams).url
            }
            is ClipprException.NetworkError -> {
                Log.e("Clippr", "Network error", e)
                null
            }
            else -> {
                Log.e("Clippr", "Error", e)
                null
            }
        }
    }
}
```

## Java Usage

```java
LinkParameters params = new LinkParameters(
    "/product/123",
    null,  // metadata
    "summer_sale",  // campaign
    "app_share",  // source
    "social",  // medium
    null,  // alias
    new SocialMetaTags("Title", "Description", "https://...")
);

Clippr.createLink(params, (shortLink, error) -> {
    if (error != null) {
        Log.e("Clippr", "Failed", error);
    } else {
        Log.d("Clippr", "URL: " + shortLink.getUrl());
    }
});
```

## Best Practices

### Cache Referral Links

```kotlin
object LinkCache {
    private val cache = mutableMapOf<String, String>()

    suspend fun getReferralLink(userId: String): String {
        cache[userId]?.let { return it }

        val url = createReferralLink(userId)
        cache[userId] = url
        return url
    }
}
```

### Use Attribution

```kotlin
val params = LinkParameters(
    path = "/product/123",
    campaign = "user_share",
    source = "share_button",
    medium = "social"
)
```

## Next Steps

- [Event Tracking](/sdks/android/event-tracking) - Track conversions
- [API Reference](/sdks/android/api-reference) - Complete API docs

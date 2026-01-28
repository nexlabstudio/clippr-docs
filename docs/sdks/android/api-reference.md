---
title: API Reference
description: Complete API reference for the Clippr Android SDK
---

# API Reference

Complete reference for the Clippr Android SDK.

## Clippr Object

Kotlin object (singleton) for all SDK operations.

### initialize

Initialize the SDK.

```kotlin
fun initialize(
    context: Context,
    apiKey: String,
    debug: Boolean = false
)

fun initialize(context: Context, config: ClipprConfig)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `context` | `Context` | Application context |
| `apiKey` | `String` | Your API key |
| `debug` | `Boolean` | Enable debug logging |

---

### getInitialLink

Get the deep link that opened the app.

```kotlin
// Suspend function
suspend fun getInitialLink(): ClipprLink?

// Callback
fun getInitialLink(callback: (ClipprLink?) -> Unit)
```

---

### onLink

Callback for runtime links.

```kotlin
var onLink: ((ClipprLink) -> Unit)?

// Java
fun setOnLink(listener: (ClipprLink) -> Unit)
```

---

### handle

Handle an incoming App Link intent.

```kotlin
fun handle(intent: Intent?): Boolean
```

Returns `true` if the intent contained a Clippr link.

---

### track

Track a custom event.

```kotlin
// Suspend
suspend fun track(eventName: String, params: Map<String, Any>? = null)

// Callback
fun track(eventName: String, params: Map<String, Any>?, callback: ((Exception?) -> Unit)?)
```

---

### trackRevenue

Track a revenue event.

```kotlin
// Suspend
suspend fun trackRevenue(
    eventName: String,
    revenue: Double,
    currency: String,
    params: Map<String, Any>? = null
)

// Callback
fun trackRevenue(
    eventName: String,
    revenue: Double,
    currency: String,
    params: Map<String, Any>?,
    callback: ((Exception?) -> Unit)?
)
```

---

### createLink

Create a short link.

```kotlin
// Suspend
suspend fun createLink(parameters: LinkParameters): ShortLink

// Callback
fun createLink(parameters: LinkParameters, callback: (ShortLink?, Exception?) -> Unit)
```

---

### isInitialized

Check if SDK is initialized.

```kotlin
val isInitialized: Boolean
```

---

## Data Classes

### ClipprLink

```kotlin
data class ClipprLink(
    val path: String,
    val metadata: Map<String, Any?>?,
    val attribution: Attribution?,
    val matchType: MatchType,
    val confidence: Double?
)
```

---

### Attribution

```kotlin
data class Attribution(
    val campaign: String?,
    val source: String?,
    val medium: String?
)
```

---

### MatchType

```kotlin
enum class MatchType {
    DIRECT,
    DETERMINISTIC,
    PROBABILISTIC,
    NONE
}
```

| Value | Description |
|-------|-------------|
| `DIRECT` | App Link with app installed |
| `DETERMINISTIC` | Install Referrer match (100%) |
| `PROBABILISTIC` | Fingerprint matching |
| `NONE` | No match |

---

### LinkParameters

```kotlin
data class LinkParameters(
    val path: String,
    val metadata: Map<String, Any?>? = null,
    val campaign: String? = null,
    val source: String? = null,
    val medium: String? = null,
    val alias: String? = null,
    val socialTags: SocialMetaTags? = null
)
```

---

### SocialMetaTags

```kotlin
data class SocialMetaTags(
    val title: String? = null,
    val description: String? = null,
    val imageUrl: String? = null
)
```

---

### ShortLink

```kotlin
data class ShortLink(
    val url: String,
    val shortCode: String,
    val path: String
)
```

---

### ClipprConfig

```kotlin
data class ClipprConfig(
    val apiKey: String,
    val debug: Boolean = false,
    val timeout: Long = 30000,
    val baseUrl: String? = null
)
```

---

## Exceptions

### ClipprException

```kotlin
sealed class ClipprException : Exception() {
    object NotInitialized : ClipprException()
    object InvalidAPIKey : ClipprException()
    object AliasAlreadyExists : ClipprException()
    data class NetworkError(val cause: Throwable) : ClipprException()
    object RateLimited : ClipprException()
}
```

---

## Java Interop

All public methods have `@JvmStatic` and `@JvmOverloads` annotations for Java compatibility.

```java
// Initialize
Clippr.initialize(context, "API_KEY", true);

// Get link
Clippr.getInitialLink(link -> {
    if (link != null) {
        String path = link.getPath();
    }
});

// Set callback
Clippr.setOnLink(link -> {
    handleDeepLink(link);
});

// Track
Clippr.track("event", params, error -> {});

// Create link
Clippr.createLink(params, (link, error) -> {});
```

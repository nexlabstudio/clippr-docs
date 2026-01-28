---
title: API Reference
description: Complete API reference for the Clippr iOS SDK
---

# API Reference

Complete reference for the Clippr iOS SDK.

## Clippr Class

The main SDK class. Uses a shared singleton instance.

### initialize

Initialize the Clippr SDK.

```swift
static func initialize(
    apiKey: String,
    debug: Bool = false,
    timeout: TimeInterval = 30
)
```

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `apiKey` | `String` | Required | Your API key from the dashboard |
| `debug` | `Bool` | `false` | Enable debug logging |
| `timeout` | `TimeInterval` | `30` | Network request timeout |

---

### getInitialLink

Get the deep link that opened the app.

```swift
// Async
static func getInitialLink() async -> ClipprLink?

// Completion handler
static func getInitialLink(completion: @escaping (ClipprLink?) -> Void)
```

**Returns:** The link data, or `nil` if no link.

---

### onLink

Callback for links received while the app is running.

```swift
static var onLink: ((ClipprLink) -> Void)?
```

---

### handleUniversalLink

Handle an incoming Universal Link.

```swift
// From URL
@discardableResult
static func handleUniversalLink(_ url: URL) -> Bool

// From NSUserActivity
@discardableResult
static func handleUniversalLink(_ userActivity: NSUserActivity) -> Bool
```

**Returns:** `true` if the link was handled.

---

### track

Track a custom event.

```swift
// Async
static func track(_ eventName: String, params: [String: Any]? = nil) async throws

// Completion handler
static func track(_ eventName: String, params: [String: Any]? = nil, completion: ((Error?) -> Void)? = nil)
```

---

### trackRevenue

Track a revenue event.

```swift
// Async
static func trackRevenue(
    _ eventName: String,
    revenue: Double,
    currency: String,
    params: [String: Any]? = nil
) async throws

// Completion handler
static func trackRevenue(
    _ eventName: String,
    revenue: Double,
    currency: String,
    params: [String: Any]? = nil,
    completion: ((Error?) -> Void)? = nil
)
```

---

### createLink

Create a short link.

```swift
// Async
static func createLink(_ parameters: LinkParameters) async throws -> ShortLink

// Completion handler
static func createLink(_ parameters: LinkParameters, completion: @escaping (ShortLink?, Error?) -> Void)
```

---

## Data Types

### ClipprLink

```swift
struct ClipprLink: Equatable, Codable {
    let path: String
    let metadata: [String: Any]?
    let attribution: Attribution?
    let matchType: MatchType
    let confidence: Double?
}
```

| Property | Type | Description |
|----------|------|-------------|
| `path` | `String` | Deep link path |
| `metadata` | `[String: Any]?` | Custom metadata |
| `attribution` | `Attribution?` | Campaign attribution |
| `matchType` | `MatchType` | How link was matched |
| `confidence` | `Double?` | Match confidence (0.0-1.0) |

---

### Attribution

```swift
struct Attribution: Equatable, Codable {
    let campaign: String?
    let source: String?
    let medium: String?
}
```

---

### MatchType

```swift
enum MatchType: String, Codable {
    case direct
    case probabilistic
    case none
}
```

| Value | Description |
|-------|-------------|
| `.direct` | Universal Link with app installed |
| `.probabilistic` | Fingerprint matching |
| `.none` | No match found |

---

### LinkParameters

```swift
struct LinkParameters {
    var path: String
    var metadata: [String: Any]?
    var campaign: String?
    var source: String?
    var medium: String?
    var alias: String?
    var socialTags: SocialMetaTags?
}
```

---

### SocialMetaTags

```swift
struct SocialMetaTags {
    var title: String?
    var description: String?
    var imageUrl: String?
}
```

---

### ShortLink

```swift
struct ShortLink {
    let url: String
    let shortCode: String
    let path: String
}
```

---

## Errors

### ClipprError

```swift
enum ClipprError: Error {
    case notInitialized
    case invalidAPIKey
    case aliasAlreadyExists
    case networkError(Error)
    case invalidResponse
    case rateLimited
}
```

| Case | Description |
|------|-------------|
| `notInitialized` | SDK not initialized |
| `invalidAPIKey` | Invalid API key |
| `aliasAlreadyExists` | Alias already taken |
| `networkError` | Network request failed |
| `invalidResponse` | Invalid server response |
| `rateLimited` | Too many requests |

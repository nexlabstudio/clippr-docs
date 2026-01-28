---
title: Creating Links
description: Create short links programmatically in your iOS app
---

# Creating Links

Create Clippr short links programmatically from your iOS app for sharing, referrals, and dynamic content.

## Basic Link Creation

```swift
import ClipprSDK

// Using async/await
func createProductLink() async throws -> String {
    let params = LinkParameters(path: "/product/123")
    let shortLink = try await Clippr.createLink(params)
    return shortLink.url
    // Returns: https://yourapp.clppr.xyz/abc123
}

// Using completion handler
func createProductLink(completion: @escaping (String?) -> Void) {
    let params = LinkParameters(path: "/product/123")
    Clippr.createLink(params) { shortLink, error in
        completion(shortLink?.url)
    }
}
```

## Full Parameter Example

```swift
let params = LinkParameters(
    // Required: Deep link path
    path: "/product/123",

    // Optional: Custom metadata
    metadata: [
        "product_name": "Premium Headphones",
        "price": 149.99,
        "color": "black"
    ],

    // Optional: Attribution
    campaign: "summer_sale",
    source: "app_share",
    medium: "social",

    // Optional: Custom short code
    alias: "summer-headphones",

    // Optional: Social preview
    socialTags: SocialMetaTags(
        title: "Check out these headphones!",
        description: "50% off for a limited time",
        imageUrl: "https://example.com/headphones.jpg"
    )
)

let shortLink = try await Clippr.createLink(params)
// Result: https://yourapp.clppr.xyz/summer-headphones
```

## Common Use Cases

### Product Sharing

```swift
func shareProduct(_ product: Product) async {
    let params = LinkParameters(
        path: "/product/\(product.id)",
        metadata: [
            "name": product.name,
            "price": product.price
        ],
        source: "product_share",
        socialTags: SocialMetaTags(
            title: product.name,
            description: product.description,
            imageUrl: product.imageUrl
        )
    )

    do {
        let shortLink = try await Clippr.createLink(params)

        // Present share sheet
        let activityVC = UIActivityViewController(
            activityItems: [shortLink.url],
            applicationActivities: nil
        )
        present(activityVC, animated: true)
    } catch {
        print("Failed to create link: \(error)")
    }
}
```

### Referral Program

```swift
func createReferralLink(for userId: String) async throws -> String {
    let params = LinkParameters(
        path: "/referral",
        metadata: [
            "referrer_id": userId,
            "reward_type": "discount",
            "reward_value": "10%"
        ],
        campaign: "referral_program",
        source: "user_invite",
        medium: "referral",
        alias: "ref-\(userId)",
        socialTags: SocialMetaTags(
            title: "Join MyApp and get 10% off!",
            description: "Your friend invited you to join",
            imageUrl: "https://example.com/referral.jpg"
        )
    )

    let shortLink = try await Clippr.createLink(params)
    return shortLink.url
}
```

### SwiftUI Share Button

```swift
struct ProductView: View {
    let product: Product
    @State private var shareURL: URL?
    @State private var showingShareSheet = false

    var body: some View {
        VStack {
            // Product details...

            Button("Share") {
                Task {
                    await createShareLink()
                }
            }
        }
        .sheet(isPresented: $showingShareSheet) {
            if let url = shareURL {
                ShareSheet(items: [url])
            }
        }
    }

    func createShareLink() async {
        let params = LinkParameters(
            path: "/product/\(product.id)",
            socialTags: SocialMetaTags(
                title: product.name,
                description: product.shortDescription,
                imageUrl: product.imageUrl
            )
        )

        do {
            let shortLink = try await Clippr.createLink(params)
            shareURL = URL(string: shortLink.url)
            showingShareSheet = true
        } catch {
            print("Error: \(error)")
        }
    }
}

struct ShareSheet: UIViewControllerRepresentable {
    let items: [Any]

    func makeUIViewController(context: Context) -> UIActivityViewController {
        UIActivityViewController(activityItems: items, applicationActivities: nil)
    }

    func updateUIViewController(_ uiViewController: UIActivityViewController, context: Context) {}
}
```

## ShortLink Response

```swift
let shortLink = try await Clippr.createLink(params)

// The full URL
print(shortLink.url)        // https://yourapp.clppr.xyz/abc123

// The short code (or custom alias)
print(shortLink.shortCode)  // abc123

// The original path
print(shortLink.path)       // /product/123
```

## Error Handling

```swift
func createLinkSafely(_ params: LinkParameters) async -> String? {
    do {
        let shortLink = try await Clippr.createLink(params)
        return shortLink.url
    } catch let error as ClipprError {
        switch error {
        case .aliasAlreadyExists:
            // Try again without custom alias
            var newParams = params
            newParams.alias = nil
            return await createLinkSafely(newParams)

        case .networkError(let underlying):
            print("Network error: \(underlying)")
            return nil

        case .invalidAPIKey:
            print("Invalid API key")
            return nil

        default:
            print("Error: \(error)")
            return nil
        }
    } catch {
        print("Unexpected error: \(error)")
        return nil
    }
}
```

## Caching Links

For links that don't change (like referral links), cache them:

```swift
class LinkCache {
    static let shared = LinkCache()
    private var cache: [String: String] = [:]

    func getReferralLink(userId: String) async throws -> String {
        let key = "referral-\(userId)"

        if let cached = cache[key] {
            return cached
        }

        let url = try await createReferralLink(for: userId)
        cache[key] = url
        return url
    }
}
```

## Best Practices

### Use Meaningful Aliases

```swift
// Good - descriptive and memorable
alias: "summer-sale-2024"
alias: "ref-\(username)"

// Avoid - not memorable
alias: "a1b2c3"
```

### Include Attribution

```swift
let params = LinkParameters(
    path: "/product/123",
    campaign: "user_share",   // What campaign
    source: "share_button",   // Where in app
    medium: "social"          // Type of share
)
```

### Optimize Social Tags

```swift
let socialTags = SocialMetaTags(
    title: "Keep under 60 chars",          // og:title
    description: "Keep under 155 chars",   // og:description
    imageUrl: "https://..."                // 1200x630px recommended
)
```

## Next Steps

- [Event Tracking](/sdks/ios/event-tracking) - Track conversions
- [API Reference](/sdks/ios/api-reference) - Complete API docs

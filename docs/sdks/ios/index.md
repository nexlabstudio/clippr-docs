---
title: iOS SDK
description: Deep linking and mobile attribution SDK for iOS
---

# iOS SDK

The Clippr iOS SDK provides deep linking and mobile attribution for native iOS apps built with Swift.

## Features

- Universal Link handling
- Deferred deep linking (attribution after install)
- Probabilistic matching via device fingerprinting
- Event and revenue tracking
- Programmatic short link creation
- Swift async/await and completion handler APIs
- Debug mode for development

## Requirements

- iOS 13.0+
- Swift 5.7+
- Xcode 14.0+

## Quick Start

```swift
import ClipprSDK

// Initialize in your App or AppDelegate
@main
struct MyApp: App {
    init() {
        Clippr.initialize(apiKey: "YOUR_API_KEY")
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
                .onOpenURL { url in
                    Clippr.handleUniversalLink(url)
                }
        }
    }
}

// Handle deep links in your view
struct ContentView: View {
    var body: some View {
        NavigationStack {
            // Your content
        }
        .task {
            if let link = await Clippr.getInitialLink() {
                handleDeepLink(link)
            }

            Clippr.onLink = { link in
                handleDeepLink(link)
            }
        }
    }

    func handleDeepLink(_ link: ClipprLink) {
        print("Path: \(link.path)")
        print("Campaign: \(link.attribution?.campaign ?? "none")")
    }
}
```

## Documentation

<Cards>
  <Card title="Installation" href="/sdks/ios/installation">
    Add the SDK via SPM or CocoaPods
  </Card>
  <Card title="Quick Start" href="/sdks/ios/quickstart">
    Get up and running in 5 minutes
  </Card>
  <Card title="Universal Links" href="/sdks/ios/universal-links">
    Configure Universal Links
  </Card>
  <Card title="Handling Links" href="/sdks/ios/handling-links">
    Handle deep links in your app
  </Card>
  <Card title="Creating Links" href="/sdks/ios/creating-links">
    Create short links programmatically
  </Card>
  <Card title="Event Tracking" href="/sdks/ios/event-tracking">
    Track events and revenue
  </Card>
  <Card title="API Reference" href="/sdks/ios/api-reference">
    Complete API documentation
  </Card>
</Cards>

## Migration from Firebase Dynamic Links

| Firebase Dynamic Links | Clippr |
|------------------------|--------|
| `DynamicLinks.dynamicLinks().handleUniversalLink()` | `Clippr.handleUniversalLink()` |
| `DynamicLinks.dynamicLinks()?.dynamicLink(fromCustomSchemeURL:)` | `Clippr.handleUniversalLink()` |

See the [Migration Guide](/guides/migration) for a complete walkthrough.

## Source Code

The iOS SDK is open source and available on GitHub:

[github.com/nexlabstudio/clippr-ios-sdk](https://github.com/nexlabstudio/clippr-ios-sdk)

## Support

- [GitHub Issues](https://github.com/nexlabstudio/clippr-ios-sdk/issues) - Bug reports and feature requests
- [FAQ](/resources/faq) - Frequently asked questions
- [Troubleshooting](/resources/troubleshooting) - Common issues and solutions

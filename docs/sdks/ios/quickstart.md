---
title: Quick Start
description: Get started with the Clippr iOS SDK in 5 minutes
---

# Quick Start

This guide will help you integrate Clippr into your iOS app and handle your first deep link.

## Prerequisites

- Clippr SDK installed ([Installation Guide](/sdks/ios/installation))
- API key from the [Clippr Dashboard](https://app.useclippr.xyz)
- Associated Domains configured

## Initialize the SDK

### SwiftUI App

```swift
import SwiftUI
import ClipprSDK

@main
struct MyApp: App {
    init() {
        Clippr.initialize(
            apiKey: "YOUR_API_KEY",
            debug: true // Set to false in production
        )
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
                .onOpenURL { url in
                    // Handle Universal Links
                    Clippr.handleUniversalLink(url)
                }
        }
    }
}
```

### UIKit with AppDelegate

```swift
import UIKit
import ClipprSDK

@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {

        Clippr.initialize(
            apiKey: "YOUR_API_KEY",
            debug: true
        )

        return true
    }
}
```

### UIKit with SceneDelegate

```swift
import UIKit
import ClipprSDK

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(
        _ scene: UIScene,
        willConnectTo session: UISceneSession,
        options connectionOptions: UIScene.ConnectionOptions
    ) {
        // Handle links from cold start
        if let userActivity = connectionOptions.userActivities.first {
            Clippr.handleUniversalLink(userActivity)
        }
    }

    func scene(_ scene: UIScene, continue userActivity: NSUserActivity) {
        // Handle links while app is running
        Clippr.handleUniversalLink(userActivity)
    }

    func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        if let url = URLContexts.first?.url {
            Clippr.handleUniversalLink(url)
        }
    }
}
```

## Handle Deep Links

### SwiftUI

```swift
import SwiftUI
import ClipprSDK

struct ContentView: View {
    @State private var deepLinkPath: String?

    var body: some View {
        NavigationStack {
            VStack {
                if let path = deepLinkPath {
                    Text("Deep link: \(path)")
                } else {
                    Text("No deep link")
                }
            }
        }
        .task {
            await initDeepLinks()
        }
    }

    func initDeepLinks() async {
        // Get the link that opened the app (direct or deferred)
        if let link = await Clippr.getInitialLink() {
            handleDeepLink(link)
        }

        // Listen for links while app is running
        Clippr.onLink = { link in
            handleDeepLink(link)
        }
    }

    func handleDeepLink(_ link: ClipprLink) {
        print("Deep link received:")
        print("  Path: \(link.path)")
        print("  Metadata: \(link.metadata ?? [:])")
        print("  Campaign: \(link.attribution?.campaign ?? "none")")
        print("  Match type: \(link.matchType)")

        deepLinkPath = link.path

        // Navigate based on path
        navigateToPath(link.path)
    }

    func navigateToPath(_ path: String) {
        if path.hasPrefix("/product/") {
            let productId = String(path.dropFirst("/product/".count))
            // Navigate to product
        } else if path.hasPrefix("/promo/") {
            let promoCode = String(path.dropFirst("/promo/".count))
            // Navigate to promo
        }
    }
}
```

### UIKit

```swift
import UIKit
import ClipprSDK

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        initDeepLinks()
    }

    func initDeepLinks() {
        // Using async/await
        Task {
            if let link = await Clippr.getInitialLink() {
                handleDeepLink(link)
            }
        }

        // Or using completion handler
        Clippr.getInitialLink { link in
            if let link = link {
                self.handleDeepLink(link)
            }
        }

        // Listen for runtime links
        Clippr.onLink = { [weak self] link in
            self?.handleDeepLink(link)
        }
    }

    func handleDeepLink(_ link: ClipprLink) {
        print("Received: \(link.path)")

        // Navigate on main thread
        DispatchQueue.main.async {
            self.navigateToPath(link.path)
        }
    }

    func navigateToPath(_ path: String) {
        // Your navigation logic
    }
}
```

## Test Your Integration

### Create a Test Link

1. Go to the [Clippr Dashboard](https://app.useclippr.xyz)
2. Navigate to **Links** → **Create Link**
3. Enter:
   - **Path**: `/test/hello`
   - **Alias**: `ios-test` (optional)
4. Click **Create**

### Test Direct Deep Link

1. Send the link to your test device (iMessage, Notes, etc.)
2. With your app installed, tap the link
3. Your app should open directly to the path

### Test Deferred Deep Link

1. Delete your app from the device
2. Tap the test link → You'll see Safari briefly, then App Store
3. Install the app via TestFlight or Xcode
4. Open the app
5. `getInitialLink()` should return the link data

<Info>
For development testing, run from Xcode after clicking the link to simulate post-install behavior.
</Info>

## Debug Mode

Enable debug logging:

```swift
Clippr.initialize(apiKey: "YOUR_API_KEY", debug: true)
```

Debug logs show:
- SDK initialization
- Link detection and matching
- API requests and responses
- Attribution data

## Complete Example

```swift
import SwiftUI
import ClipprSDK

@main
struct MyApp: App {
    init() {
        Clippr.initialize(apiKey: "YOUR_API_KEY", debug: true)
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

struct ContentView: View {
    @State private var lastLink: ClipprLink?

    var body: some View {
        VStack(spacing: 20) {
            Text("Clippr iOS Demo")
                .font(.title)

            if let link = lastLink {
                VStack(alignment: .leading) {
                    Text("Path: \(link.path)")
                    Text("Match: \(String(describing: link.matchType))")
                    if let campaign = link.attribution?.campaign {
                        Text("Campaign: \(campaign)")
                    }
                }
                .padding()
                .background(Color.gray.opacity(0.1))
                .cornerRadius(8)
            } else {
                Text("No deep link received")
                    .foregroundColor(.secondary)
            }
        }
        .padding()
        .task {
            if let link = await Clippr.getInitialLink() {
                lastLink = link
            }

            Clippr.onLink = { link in
                lastLink = link
            }
        }
    }
}
```

## Next Steps

- [Universal Links](/sdks/ios/universal-links) - Deep dive into Universal Links configuration
- [Handling Links](/sdks/ios/handling-links) - Advanced link handling patterns
- [Creating Links](/sdks/ios/creating-links) - Create short links in your app

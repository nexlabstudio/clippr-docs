---
title: Handling Links
description: Handle deep links in your iOS app
---

# Handling Links

Learn how to properly handle deep links in your iOS app.

## Getting the Initial Link

The initial link is the link that opened your app or was matched via deferred deep linking:

```swift
// Using async/await
func initDeepLinks() async {
    if let link = await Clippr.getInitialLink() {
        print("App opened with: \(link.path)")
        handleDeepLink(link)
    } else {
        print("Organic launch (no deep link)")
    }
}

// Using completion handler
func initDeepLinks() {
    Clippr.getInitialLink { link in
        if let link = link {
            self.handleDeepLink(link)
        }
    }
}
```

<Info>
`getInitialLink()` should only be called once per app launch. The SDK caches the result internally.
</Info>

## Listening for Runtime Links

Handle links received while your app is already running:

```swift
// Set up the listener (usually in viewDidLoad or onAppear)
Clippr.onLink = { [weak self] link in
    self?.handleDeepLink(link)
}

// Clean up when done (optional)
deinit {
    Clippr.onLink = nil
}
```

## Processing Link Data

### ClipprLink Properties

```swift
func handleDeepLink(_ link: ClipprLink) {
    // The deep link path
    let path = link.path  // e.g., "/product/123"

    // Custom metadata
    if let metadata = link.metadata {
        let discount = metadata["discount"] as? String
        let referrer = metadata["referrer"] as? String
    }

    // Attribution data
    if let attribution = link.attribution {
        print("Campaign: \(attribution.campaign ?? "none")")
        print("Source: \(attribution.source ?? "none")")
        print("Medium: \(attribution.medium ?? "none")")
    }

    // Match type
    switch link.matchType {
    case .direct:
        print("Direct link (app was installed)")
    case .probabilistic:
        print("Probabilistic match")
        if let confidence = link.confidence {
            print("Confidence: \(confidence)")
        }
    case .none:
        print("No match")
    }
}
```

## Navigation Patterns

### SwiftUI with NavigationStack

```swift
import SwiftUI
import ClipprSDK

struct ContentView: View {
    @State private var navigationPath = NavigationPath()

    var body: some View {
        NavigationStack(path: $navigationPath) {
            HomeView()
                .navigationDestination(for: String.self) { path in
                    destinationView(for: path)
                }
        }
        .task {
            await initDeepLinks()
        }
    }

    func initDeepLinks() async {
        if let link = await Clippr.getInitialLink() {
            navigateTo(link.path)
        }

        Clippr.onLink = { link in
            navigateTo(link.path)
        }
    }

    func navigateTo(_ path: String) {
        // Reset navigation and push new path
        navigationPath = NavigationPath()
        navigationPath.append(path)
    }

    @ViewBuilder
    func destinationView(for path: String) -> some View {
        if path.hasPrefix("/product/") {
            let id = String(path.dropFirst("/product/".count))
            ProductView(productId: id)
        } else if path.hasPrefix("/category/") {
            let category = String(path.dropFirst("/category/".count))
            CategoryView(category: category)
        } else {
            Text("Unknown path: \(path)")
        }
    }
}
```

### UIKit with Navigation Controller

```swift
class MainViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        Task {
            if let link = await Clippr.getInitialLink() {
                handleDeepLink(link)
            }
        }

        Clippr.onLink = { [weak self] link in
            self?.handleDeepLink(link)
        }
    }

    func handleDeepLink(_ link: ClipprLink) {
        DispatchQueue.main.async {
            self.navigateToPath(link.path)
        }
    }

    func navigateToPath(_ path: String) {
        // Pop to root first
        navigationController?.popToRootViewController(animated: false)

        // Navigate based on path
        if path.hasPrefix("/product/") {
            let productId = String(path.dropFirst("/product/".count))
            let vc = ProductViewController(productId: productId)
            navigationController?.pushViewController(vc, animated: true)
        } else if path.hasPrefix("/settings") {
            let vc = SettingsViewController()
            navigationController?.pushViewController(vc, animated: true)
        }
    }
}
```

### With Coordinators

```swift
protocol DeepLinkHandler: AnyObject {
    func handleDeepLink(_ link: ClipprLink)
}

class AppCoordinator: DeepLinkHandler {
    private let window: UIWindow
    private var childCoordinators: [Any] = []

    init(window: UIWindow) {
        self.window = window
        setupDeepLinks()
    }

    func setupDeepLinks() {
        Task {
            if let link = await Clippr.getInitialLink() {
                handleDeepLink(link)
            }
        }

        Clippr.onLink = { [weak self] link in
            self?.handleDeepLink(link)
        }
    }

    func handleDeepLink(_ link: ClipprLink) {
        // Route to appropriate coordinator
        let path = link.path

        if path.hasPrefix("/product/") {
            showProductFlow(path: path, attribution: link.attribution)
        } else if path.hasPrefix("/checkout") {
            showCheckoutFlow(metadata: link.metadata)
        }
    }

    private func showProductFlow(path: String, attribution: Attribution?) {
        // Create and start product coordinator
    }
}
```

## Handling Match Types

```swift
func handleDeepLink(_ link: ClipprLink) {
    switch link.matchType {
    case .direct:
        // 100% confidence - direct Universal Link click
        navigateAndTrack(link)

    case .probabilistic:
        // Check confidence for probabilistic matches
        let confidence = link.confidence ?? 0

        if confidence > 0.8 {
            // High confidence - navigate directly
            navigateAndTrack(link)
        } else if confidence > 0.5 {
            // Medium confidence - maybe confirm
            showConfirmationIfNeeded(link)
        } else {
            // Low confidence - might be wrong user
            handleLowConfidenceMatch(link)
        }

    case .none:
        // Shouldn't happen for received links
        break
    }
}

func showConfirmationIfNeeded(_ link: ClipprLink) {
    // Optionally ask user to confirm the deep link destination
    // "Were you looking for [Product Name]?"
}
```

## Delayed Navigation

Wait for app initialization before navigating:

```swift
class AppState: ObservableObject {
    @Published var isReady = false
    private var pendingLink: ClipprLink?

    func initialize() async {
        // Load user data, configure services, etc.
        await loadUserData()
        await configureAnalytics()

        isReady = true

        // Process pending link
        if let link = pendingLink {
            handleDeepLink(link)
            pendingLink = nil
        }
    }

    func receivedLink(_ link: ClipprLink) {
        if isReady {
            handleDeepLink(link)
        } else {
            pendingLink = link
        }
    }
}
```

## Error Handling

```swift
func initDeepLinks() async {
    do {
        if let link = await Clippr.getInitialLink() {
            handleDeepLink(link)
        }
    } catch {
        print("Error getting initial link: \(error)")
        // Continue without link - don't block app launch
    }
}

func handleDeepLink(_ link: ClipprLink) {
    do {
        try navigateToPath(link.path)
    } catch {
        print("Navigation failed: \(error)")
        // Navigate to home as fallback
        navigateToHome()
    }
}
```

## Next Steps

- [Creating Links](/sdks/ios/creating-links) - Generate short links in your app
- [Event Tracking](/sdks/ios/event-tracking) - Track conversions from deep links
- [API Reference](/sdks/ios/api-reference) - Complete API documentation

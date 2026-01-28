---
title: Handling Links
description: Handle deep links in your Flutter app
---

# Handling Links

Learn how to properly handle deep links in different scenarios.

## Link Types

### Direct Links
When a user clicks a Clippr link and your app is already installed, the app opens directly via Universal Link (iOS) or App Link (Android).

### Deferred Links
When a user clicks a link but your app is not installed, they're redirected to the app store. After installation, the SDK retrieves the original link data.

## Getting the Initial Link

The initial link is the link that opened or was associated with your app launch:

```dart
Future<void> _initDeepLinks() async {
  final link = await Clippr.getInitialLink();

  if (link != null) {
    print('App opened with link: ${link.path}');
    _handleDeepLink(link);
  } else {
    print('App opened without a link (organic)');
  }
}
```

<Info>
`getInitialLink()` should only be called once per app launch. The SDK internally caches the result and clears it after the first call for deferred links.
</Info>

## Listening for Runtime Links

Handle links received while your app is already running:

```dart
@override
void initState() {
  super.initState();

  // Set up the link listener
  Clippr.onLink = (ClipprLink link) {
    print('Received link while app running: ${link.path}');
    _handleDeepLink(link);
  };
}

@override
void dispose() {
  // Clean up (optional - SDK handles this)
  Clippr.onLink = null;
  super.dispose();
}
```

## Handling Link Data

### ClipprLink Properties

```dart
void _handleDeepLink(ClipprLink link) {
  // The deep link path (e.g., "/product/123")
  final path = link.path;

  // Custom metadata attached to the link
  final metadata = link.metadata;
  // metadata might contain: {"discount": "20%", "referrer": "user456"}

  // Attribution data
  final attribution = link.attribution;
  if (attribution != null) {
    print('Campaign: ${attribution.campaign}');
    print('Source: ${attribution.source}');
    print('Medium: ${attribution.medium}');
  }

  // How the link was matched
  final matchType = link.matchType;
  // MatchType.direct - clicked with app installed
  // MatchType.deterministic - matched via Install Referrer
  // MatchType.probabilistic - matched via fingerprinting
  // MatchType.none - no match (shouldn't happen for received links)

  // Confidence score for probabilistic matches (0.0 - 1.0)
  final confidence = link.confidence;
}
```

### Using Metadata

Links can carry custom metadata that you define when creating the link:

```dart
void _handleDeepLink(ClipprLink link) {
  final metadata = link.metadata;

  if (metadata != null) {
    // Access metadata values
    final discount = metadata['discount'] as String?;
    final referrer = metadata['referrer'] as String?;
    final productData = metadata['product'] as Map<String, dynamic>?;

    if (discount != null) {
      _applyDiscount(discount);
    }

    if (referrer != null) {
      _trackReferral(referrer);
    }
  }
}
```

## Navigation Patterns

### Simple Path-Based Navigation

```dart
void _handleDeepLink(ClipprLink link) {
  final path = link.path;

  if (path == '/') {
    _navigateToHome();
  } else if (path.startsWith('/product/')) {
    final productId = path.replaceFirst('/product/', '');
    _navigateToProduct(productId);
  } else if (path.startsWith('/category/')) {
    final category = path.replaceFirst('/category/', '');
    _navigateToCategory(category);
  } else if (path == '/cart') {
    _navigateToCart();
  } else {
    // Unknown path - navigate to home or show error
    _navigateToHome();
  }
}
```

### With GoRouter

```dart
void _handleDeepLink(ClipprLink link) {
  // Store attribution for later tracking
  if (link.attribution != null) {
    _attributionService.setAttribution(link.attribution!);
  }

  // Navigate using GoRouter
  context.go(link.path);
}
```

### With Navigator 2.0

```dart
class MyRouterDelegate extends RouterDelegate<String> {
  String? _currentPath;
  ClipprLink? _pendingLink;

  void handleDeepLink(ClipprLink link) {
    _pendingLink = link;
    _currentPath = link.path;
    notifyListeners();
  }

  @override
  Widget build(BuildContext context) {
    return Navigator(
      pages: _buildPages(),
      onPopPage: (route, result) {
        // Handle back navigation
        return route.didPop(result);
      },
    );
  }

  List<Page> _buildPages() {
    // Build pages based on _currentPath
    return [
      MaterialPage(child: HomePage()),
      if (_currentPath?.startsWith('/product/') == true)
        MaterialPage(
          child: ProductPage(
            productId: _currentPath!.replaceFirst('/product/', ''),
            link: _pendingLink,
          ),
        ),
    ];
  }
}
```

## Handling Match Types

Different match types have different confidence levels:

```dart
void _handleDeepLink(ClipprLink link) {
  switch (link.matchType) {
    case MatchType.direct:
      // 100% confidence - user clicked with app installed
      _navigateAndTrack(link, confident: true);
      break;

    case MatchType.deterministic:
      // 100% confidence - matched via Install Referrer (Android)
      _navigateAndTrack(link, confident: true);
      break;

    case MatchType.probabilistic:
      // Variable confidence - check the score
      final confidence = link.confidence ?? 0.0;
      if (confidence > 0.8) {
        // High confidence - navigate directly
        _navigateAndTrack(link, confident: true);
      } else if (confidence > 0.5) {
        // Medium confidence - maybe show confirmation
        _showLinkConfirmation(link);
      } else {
        // Low confidence - might be wrong user
        _handleLowConfidenceMatch(link);
      }
      break;

    case MatchType.none:
      // Shouldn't happen for received links
      break;
  }
}
```

## Delayed Navigation

Sometimes you need to delay navigation until your app is ready:

```dart
class _MyAppState extends State<MyApp> {
  ClipprLink? _pendingLink;
  bool _isAppReady = false;

  @override
  void initState() {
    super.initState();
    _initDeepLinks();
    _initApp();
  }

  Future<void> _initApp() async {
    // Initialize services, load user data, etc.
    await _authService.initialize();
    await _userService.loadUser();

    setState(() => _isAppReady = true);

    // Process pending link now that app is ready
    if (_pendingLink != null) {
      _handleDeepLink(_pendingLink!);
      _pendingLink = null;
    }
  }

  Future<void> _initDeepLinks() async {
    final link = await Clippr.getInitialLink();
    if (link != null) {
      if (_isAppReady) {
        _handleDeepLink(link);
      } else {
        _pendingLink = link;
      }
    }

    Clippr.onLink = (link) {
      if (_isAppReady) {
        _handleDeepLink(link);
      } else {
        _pendingLink = link;
      }
    };
  }
}
```

## Manual Link Handling

For custom URL scheme links or when you need manual control:

```dart
// Handle a URL manually
final uri = Uri.parse('https://yourapp.clppr.xyz/product/123');
final handled = await Clippr.handle(uri);

if (handled) {
  print('Link was handled by Clippr');
} else {
  print('Link was not a Clippr link');
}
```

## Error Handling

```dart
Future<void> _initDeepLinks() async {
  try {
    final link = await Clippr.getInitialLink();
    if (link != null) {
      _handleDeepLink(link);
    }
  } catch (e) {
    print('Error getting initial link: $e');
    // Continue without link - don't block app startup
  }
}

void _handleDeepLink(ClipprLink link) {
  try {
    _navigateToPath(link.path);
  } catch (e) {
    print('Error handling deep link: $e');
    // Navigate to home as fallback
    _navigateToHome();
  }
}
```

## Next Steps

- [Creating Links](/sdks/flutter/creating-links) - Generate short links in your app
- [Event Tracking](/sdks/flutter/event-tracking) - Track conversions from deep links
- [API Reference](/sdks/flutter/api-reference) - Complete API documentation

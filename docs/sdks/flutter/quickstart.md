---
title: Quick Start
description: Get started with the Clippr Flutter SDK in 5 minutes
---

# Quick Start

This guide will help you integrate Clippr into your Flutter app and handle your first deep link.

## Prerequisites

- Clippr SDK installed ([Installation Guide](/sdks/flutter/installation))
- API key from the [Clippr Dashboard](https://app.useclippr.xyz)
- Platform setup complete (Associated Domains for iOS, Intent Filter for Android)

## Initialize the SDK

Initialize Clippr as early as possible in your app, typically in `main()`:

```dart
import 'package:flutter/material.dart';
import 'package:clippr/clippr.dart';

void main() async {
  // Required for async initialization before runApp
  WidgetsFlutterBinding.ensureInitialized();

  // Initialize Clippr SDK
  await Clippr.initialize(
    apiKey: 'YOUR_API_KEY',
    debug: true, // Set to false in production
  );

  runApp(MyApp());
}
```

<Warning>
Always call `WidgetsFlutterBinding.ensureInitialized()` before initializing Clippr.
</Warning>

## Handle Deep Links

There are two ways your app receives deep links:

1. **Initial Link**: The link that opened your app (cold start or deferred)
2. **Runtime Links**: Links received while your app is already running

### Basic Implementation

```dart
class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  @override
  void initState() {
    super.initState();
    _initDeepLinks();
  }

  Future<void> _initDeepLinks() async {
    // 1. Get the link that opened the app
    final link = await Clippr.getInitialLink();
    if (link != null) {
      _handleDeepLink(link);
    }

    // 2. Listen for links while app is running
    Clippr.onLink = (link) {
      _handleDeepLink(link);
    };
  }

  void _handleDeepLink(ClipprLink link) {
    print('Received deep link:');
    print('  Path: ${link.path}');
    print('  Metadata: ${link.metadata}');
    print('  Campaign: ${link.attribution?.campaign}');
    print('  Match Type: ${link.matchType}');

    // Navigate based on the path
    _navigateToPath(link.path);
  }

  void _navigateToPath(String path) {
    // Implement your navigation logic
    if (path.startsWith('/product/')) {
      final productId = path.split('/').last;
      // Navigate to product page
    } else if (path.startsWith('/promo/')) {
      final promoCode = path.split('/').last;
      // Navigate to promo page
    }
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // ...
    );
  }
}
```

## With GoRouter

If you're using GoRouter for navigation:

```dart
import 'package:go_router/go_router.dart';
import 'package:clippr/clippr.dart';

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  late final GoRouter _router;

  @override
  void initState() {
    super.initState();
    _router = _createRouter();
    _initDeepLinks();
  }

  GoRouter _createRouter() {
    return GoRouter(
      routes: [
        GoRoute(path: '/', builder: (context, state) => HomePage()),
        GoRoute(path: '/product/:id', builder: (context, state) {
          final id = state.pathParameters['id']!;
          return ProductPage(productId: id);
        }),
        GoRoute(path: '/promo/:code', builder: (context, state) {
          final code = state.pathParameters['code']!;
          return PromoPage(promoCode: code);
        }),
      ],
    );
  }

  Future<void> _initDeepLinks() async {
    final link = await Clippr.getInitialLink();
    if (link != null) {
      _handleDeepLink(link);
    }

    Clippr.onLink = (link) {
      _handleDeepLink(link);
    };
  }

  void _handleDeepLink(ClipprLink link) {
    // GoRouter will match the path to the correct route
    _router.go(link.path);
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: _router,
    );
  }
}
```

## Test Your Integration

### Create a Test Link

1. Go to the [Clippr Dashboard](https://app.useclippr.xyz)
2. Navigate to **Links** → **Create Link**
3. Enter:
   - **Path**: `/product/test123`
   - **Alias**: `test-link` (optional)
4. Click **Create**

### Test Direct Deep Link

1. Send the link to your test device
2. With your app installed, click the link
3. Your app should open directly to the path

### Test Deferred Deep Link

1. Uninstall your app
2. Click the test link → You'll be redirected to the app store
3. Install the app (or run from IDE)
4. Open the app
5. The `getInitialLink()` should return the link data

<Info>
During development, run from your IDE after clicking the link to simulate the post-install flow.
</Info>

## Debug Mode

Enable debug logging to troubleshoot issues:

```dart
await Clippr.initialize(
  apiKey: 'YOUR_API_KEY',
  debug: true,
);
```

Debug logs show:
- SDK initialization status
- Link detection and matching
- API requests and responses
- Attribution data

## Complete Example

Here's a complete example with error handling:

```dart
import 'package:flutter/material.dart';
import 'package:clippr/clippr.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  try {
    await Clippr.initialize(
      apiKey: 'YOUR_API_KEY',
      debug: true,
    );
  } catch (e) {
    print('Failed to initialize Clippr: $e');
  }

  runApp(MyApp());
}

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  ClipprLink? _lastLink;

  @override
  void initState() {
    super.initState();
    _initDeepLinks();
  }

  Future<void> _initDeepLinks() async {
    try {
      final link = await Clippr.getInitialLink();
      if (link != null) {
        setState(() => _lastLink = link);
        _handleDeepLink(link);
      }
    } catch (e) {
      print('Error getting initial link: $e');
    }

    Clippr.onLink = (link) {
      setState(() => _lastLink = link);
      _handleDeepLink(link);
    };
  }

  void _handleDeepLink(ClipprLink link) {
    // Your navigation logic here
    print('Navigating to: ${link.path}');
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Clippr Demo')),
        body: Center(
          child: _lastLink != null
              ? Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Text('Last Link:'),
                    Text('Path: ${_lastLink!.path}'),
                    Text('Match: ${_lastLink!.matchType}'),
                  ],
                )
              : Text('No deep link received'),
        ),
      ),
    );
  }
}
```

## Next Steps

- [Handling Links](/sdks/flutter/handling-links) - Advanced link handling patterns
- [Creating Links](/sdks/flutter/creating-links) - Create short links in your app
- [Event Tracking](/sdks/flutter/event-tracking) - Track conversions and revenue

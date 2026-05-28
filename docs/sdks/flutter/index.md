---
title: Flutter SDK
description: Deep linking and mobile attribution SDK for Flutter
---

# Flutter SDK

The Flutter integration of Clippr's deep linking and mobile attribution platform.

## Features

- Deep link handling for iOS and Android
- Deferred deep linking (attribution after install)
- Deterministic matching via Install Referrer (Android)
- Probabilistic matching via device fingerprinting
- Event and revenue tracking
- Programmatic short link creation
- Debug mode for development

## Requirements

- Flutter 3.10+
- Dart 3.10+
- iOS 13.0+
- Android API 21+

## Quick Start

```dart
import 'package:clippr/clippr.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // Initialize SDK
  await Clippr.initialize(apiKey: 'YOUR_API_KEY');

  runApp(MyApp());
}

class _MyAppState extends State<MyApp> {
  @override
  void initState() {
    super.initState();
    _initDeepLinks();
  }

  Future<void> _initDeepLinks() async {
    // Get the link that opened the app (direct or deferred)
    final link = await Clippr.getInitialLink();
    if (link != null) {
      _handleDeepLink(link);
    }

    // Listen for links while app is running
    Clippr.onLink = (link) {
      _handleDeepLink(link);
    };
  }

  void _handleDeepLink(ClipprLink link) {
    print('Path: ${link.path}');
    print('Campaign: ${link.attribution?.campaign}');
    // Navigate based on path
  }
}
```

## Documentation

<Cards>
  <Card title="Installation" href="/sdks/flutter/installation">
    Add the SDK to your Flutter project
  </Card>
  <Card title="Quick Start" href="/sdks/flutter/quickstart">
    Get up and running in 5 minutes
  </Card>
  <Card title="Handling Links" href="/sdks/flutter/handling-links">
    Handle deep links in your app
  </Card>
  <Card title="Creating Links" href="/sdks/flutter/creating-links">
    Create short links programmatically
  </Card>
  <Card title="Event Tracking" href="/sdks/flutter/event-tracking">
    Track events and revenue
  </Card>
  <Card title="API Reference" href="/sdks/flutter/api-reference">
    Complete API documentation
  </Card>
</Cards>

<!-- Migration from Firebase Dynamic Links section disabled — Clippr positioning is attribution-first, not FDL replacement.
## Migration from Firebase Dynamic Links

Clippr's API is intentionally similar to Firebase Dynamic Links for easy migration:

| Firebase Dynamic Links | Clippr |
|------------------------|--------|
| `FirebaseDynamicLinks.instance.getInitialLink()` | `Clippr.getInitialLink()` |
| `FirebaseDynamicLinks.instance.onLink` | `Clippr.onLink` |
| `DynamicLinkParameters` | `LinkParameters` |
| `ShortDynamicLink` | `ShortLink` |

See the [Migration Guide](/guides/migration) for a complete walkthrough.
-->

## Source Code

The Flutter SDK is open source and available on GitHub:

[github.com/nexlabstudio/clippr-flutter](https://github.com/nexlabstudio/clippr-flutter)

## Support

- [GitHub Issues](https://github.com/nexlabstudio/clippr-flutter/issues) - Bug reports and feature requests
- [FAQ](/resources/faq) - Frequently asked questions
- [Troubleshooting](/resources/troubleshooting) - Common issues and solutions

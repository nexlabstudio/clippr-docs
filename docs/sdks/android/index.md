---
title: Android SDK
description: Deep linking and mobile attribution SDK for Android
---

# Android SDK

The Clippr Android SDK provides deep linking and mobile attribution for native Android apps built with Kotlin or Java.

## Features

- App Link handling
- Deferred deep linking (attribution after install)
- Deterministic matching via Install Referrer (100% accuracy)
- Probabilistic matching via device fingerprinting
- Event and revenue tracking
- Programmatic short link creation
- Kotlin coroutines and Java callback APIs
- Debug mode for development

## Requirements

- Android API 21+ (Android 5.0 Lollipop)
- Kotlin 1.9+ or Java 8+
- Gradle 8.0+

## Quick Start

```kotlin
import xyz.useclippr.sdk.Clippr
import xyz.useclippr.sdk.models.ClipprLink

// Initialize in Application class
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        Clippr.initialize(
            context = this,
            apiKey = "YOUR_API_KEY",
            debug = BuildConfig.DEBUG
        )
    }
}

// Handle deep links in Activity
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Handle App Link that opened this activity
        Clippr.handle(intent)

        // Get initial link (direct or deferred)
        lifecycleScope.launch {
            Clippr.getInitialLink()?.let { link ->
                handleDeepLink(link)
            }
        }

        // Listen for runtime links
        Clippr.onLink = { link ->
            handleDeepLink(link)
        }
    }

    private fun handleDeepLink(link: ClipprLink) {
        Log.d("Clippr", "Path: ${link.path}")
        Log.d("Clippr", "Campaign: ${link.attribution?.campaign}")
    }
}
```

## Documentation

<Cards>
  <Card title="Installation" href="/sdks/android/installation">
    Add the SDK via Maven or JitPack
  </Card>
  <Card title="Quick Start" href="/sdks/android/quickstart">
    Get up and running in 5 minutes
  </Card>
  <Card title="App Links" href="/sdks/android/app-links">
    Configure Android App Links
  </Card>
  <Card title="Handling Links" href="/sdks/android/handling-links">
    Handle deep links in your app
  </Card>
  <Card title="Creating Links" href="/sdks/android/creating-links">
    Create short links programmatically
  </Card>
  <Card title="Event Tracking" href="/sdks/android/event-tracking">
    Track events and revenue
  </Card>
  <Card title="API Reference" href="/sdks/android/api-reference">
    Complete API documentation
  </Card>
</Cards>

## Java Support

The SDK is fully compatible with Java:

```java
import xyz.useclippr.sdk.Clippr;

// Initialize
Clippr.initialize(context, "YOUR_API_KEY", true);

// Get initial link
Clippr.getInitialLink(link -> {
    if (link != null) {
        Log.d("Clippr", "Path: " + link.getPath());
    }
});

// Listen for links
Clippr.setOnLink(link -> {
    handleDeepLink(link);
});
```

## Source Code

The Android SDK is open source and available on GitHub:

[github.com/nexlabstudio/clippr-android-sdk](https://github.com/nexlabstudio/clippr-android-sdk)

## Support

- [GitHub Issues](https://github.com/nexlabstudio/clippr-android-sdk/issues) - Bug reports and feature requests
- [FAQ](/resources/faq) - Frequently asked questions
- [Troubleshooting](/resources/troubleshooting) - Common issues and solutions

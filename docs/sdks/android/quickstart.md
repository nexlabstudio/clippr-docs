---
title: Quick Start
description: Get started with the Clippr Android SDK in 5 minutes
---

# Quick Start

This guide will help you integrate Clippr into your Android app and handle your first deep link.

## Prerequisites

- Clippr SDK installed ([Installation Guide](/sdks/android/installation))
- API key from the [Clippr Dashboard](https://app.useclippr.xyz)
- App Links configured in AndroidManifest.xml

## Initialize the SDK

Create or update your Application class:

```kotlin
import android.app.Application
import xyz.useclippr.sdk.Clippr

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
```

Register it in `AndroidManifest.xml`:

```xml
<application
    android:name=".MyApplication"
    ...>
```

## Handle Deep Links

### Kotlin with Coroutines

```kotlin
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.launch
import xyz.useclippr.sdk.Clippr
import xyz.useclippr.sdk.models.ClipprLink

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Handle App Link that opened this activity
        Clippr.handle(intent)

        // Get the initial link (direct or deferred)
        lifecycleScope.launch {
            val link = Clippr.getInitialLink()
            if (link != null) {
                handleDeepLink(link)
            }
        }

        // Listen for links while app is running
        Clippr.onLink = { link ->
            handleDeepLink(link)
        }
    }

    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        // Handle links when activity is already open
        Clippr.handle(intent)
    }

    private fun handleDeepLink(link: ClipprLink) {
        Log.d("Clippr", "Deep link received:")
        Log.d("Clippr", "  Path: ${link.path}")
        Log.d("Clippr", "  Metadata: ${link.metadata}")
        Log.d("Clippr", "  Campaign: ${link.attribution?.campaign}")
        Log.d("Clippr", "  Match type: ${link.matchType}")

        // Navigate based on path
        navigateToPath(link.path)
    }

    private fun navigateToPath(path: String) {
        when {
            path.startsWith("/product/") -> {
                val productId = path.substringAfterLast("/")
                // Navigate to product
            }
            path.startsWith("/promo/") -> {
                val promoCode = path.substringAfterLast("/")
                // Navigate to promo
            }
            else -> {
                // Navigate to home
            }
        }
    }
}
```

### Java

```java
import xyz.useclippr.sdk.Clippr;
import xyz.useclippr.sdk.models.ClipprLink;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Handle App Link
        Clippr.handle(getIntent());

        // Get initial link
        Clippr.getInitialLink(link -> {
            if (link != null) {
                handleDeepLink(link);
            }
        });

        // Listen for runtime links
        Clippr.setOnLink(link -> {
            handleDeepLink(link);
        });
    }

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        Clippr.handle(intent);
    }

    private void handleDeepLink(ClipprLink link) {
        Log.d("Clippr", "Path: " + link.getPath());

        runOnUiThread(() -> {
            navigateToPath(link.getPath());
        });
    }
}
```

### With Jetpack Compose

```kotlin
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.runtime.*
import androidx.lifecycle.lifecycleScope
import kotlinx.coroutines.launch
import xyz.useclippr.sdk.Clippr
import xyz.useclippr.sdk.models.ClipprLink

class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        Clippr.handle(intent)

        setContent {
            var lastLink by remember { mutableStateOf<ClipprLink?>(null) }

            LaunchedEffect(Unit) {
                lastLink = Clippr.getInitialLink()

                Clippr.onLink = { link ->
                    lastLink = link
                }
            }

            MyApp(
                lastLink = lastLink,
                onNavigate = { path -> navigateToPath(path) }
            )
        }
    }

    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        Clippr.handle(intent)
    }
}

@Composable
fun MyApp(lastLink: ClipprLink?, onNavigate: (String) -> Unit) {
    // Your compose UI
    lastLink?.let { link ->
        LaunchedEffect(link) {
            onNavigate(link.path)
        }
    }
}
```

## Test Your Integration

### Create a Test Link

1. Go to the [Clippr Dashboard](https://app.useclippr.xyz)
2. Create a link with path `/test/hello`
3. Copy the short URL

### Test Direct Deep Link

1. Send the link to your test device
2. With your app installed, tap the link
3. Your app should open directly

### Test Deferred Deep Link

1. Uninstall the app
2. Tap the link → Redirected to Play Store
3. Install via Android Studio
4. Open the app
5. `getInitialLink()` should return the link data

<Info>
The Android SDK uses Install Referrer for 100% accurate attribution when users install from Play Store after clicking a link.
</Info>

## Debug Mode

Enable debug logging:

```kotlin
Clippr.initialize(
    context = this,
    apiKey = "YOUR_API_KEY",
    debug = true
)
```

Check Logcat with tag `Clippr` to see:
- SDK initialization
- Link detection and matching
- Install Referrer data
- API requests

## Complete Example

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        Clippr.initialize(this, "YOUR_API_KEY", BuildConfig.DEBUG)
    }
}

class MainActivity : AppCompatActivity() {
    private val binding by lazy { ActivityMainBinding.inflate(layoutInflater) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)

        Clippr.handle(intent)

        lifecycleScope.launch {
            Clippr.getInitialLink()?.let { showLink(it) }
        }

        Clippr.onLink = { link -> showLink(link) }
    }

    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        Clippr.handle(intent)
    }

    private fun showLink(link: ClipprLink) {
        runOnUiThread {
            binding.pathText.text = "Path: ${link.path}"
            binding.matchText.text = "Match: ${link.matchType}"
            link.attribution?.campaign?.let {
                binding.campaignText.text = "Campaign: $it"
            }
        }
    }
}
```

## Next Steps

- [App Links](/sdks/android/app-links) - Deep dive into App Links
- [Handling Links](/sdks/android/handling-links) - Advanced patterns
- [Creating Links](/sdks/android/creating-links) - Generate short links

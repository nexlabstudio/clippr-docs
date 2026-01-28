---
title: Handling Links
description: Handle deep links in your Android app
---

# Handling Links

Learn how to properly handle deep links in your Android app.

## Getting the Initial Link

```kotlin
// Using coroutines
lifecycleScope.launch {
    val link = Clippr.getInitialLink()
    if (link != null) {
        Log.d("Clippr", "App opened with: ${link.path}")
        handleDeepLink(link)
    } else {
        Log.d("Clippr", "Organic launch")
    }
}

// Using callback (Java-friendly)
Clippr.getInitialLink { link ->
    if (link != null) {
        handleDeepLink(link)
    }
}
```

<Info>
The Android SDK uses Install Referrer for 100% accurate attribution when the app is installed from Play Store after clicking a link.
</Info>

## Listening for Runtime Links

```kotlin
// Set up listener
Clippr.onLink = { link ->
    handleDeepLink(link)
}

// Or in Java
Clippr.setOnLink(link -> {
    handleDeepLink(link);
});
```

## Processing Link Data

### ClipprLink Properties

```kotlin
fun handleDeepLink(link: ClipprLink) {
    // The deep link path
    val path = link.path  // e.g., "/product/123"

    // Custom metadata
    link.metadata?.let { metadata ->
        val discount = metadata["discount"] as? String
        val referrer = metadata["referrer"] as? String
    }

    // Attribution data
    link.attribution?.let { attr ->
        Log.d("Clippr", "Campaign: ${attr.campaign}")
        Log.d("Clippr", "Source: ${attr.source}")
        Log.d("Clippr", "Medium: ${attr.medium}")
    }

    // Match type
    when (link.matchType) {
        MatchType.DIRECT -> Log.d("Clippr", "Direct App Link")
        MatchType.DETERMINISTIC -> Log.d("Clippr", "Install Referrer match (100%)")
        MatchType.PROBABILISTIC -> Log.d("Clippr", "Fingerprint match: ${link.confidence}")
        MatchType.NONE -> Log.d("Clippr", "No match")
    }
}
```

## Navigation Patterns

### Simple Path Routing

```kotlin
private fun handleDeepLink(link: ClipprLink) {
    val path = link.path

    when {
        path == "/" -> navigateToHome()
        path.startsWith("/product/") -> {
            val productId = path.substringAfterLast("/")
            navigateToProduct(productId)
        }
        path.startsWith("/category/") -> {
            val category = path.substringAfterLast("/")
            navigateToCategory(category)
        }
        path == "/cart" -> navigateToCart()
        else -> navigateToHome()
    }
}
```

### With Navigation Component

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var navController: NavController

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        navController = navHostFragment.navController

        Clippr.handle(intent)

        lifecycleScope.launch {
            Clippr.getInitialLink()?.let { navigateFromLink(it) }
        }

        Clippr.onLink = { link -> navigateFromLink(link) }
    }

    private fun navigateFromLink(link: ClipprLink) {
        runOnUiThread {
            when {
                link.path.startsWith("/product/") -> {
                    val id = link.path.substringAfterLast("/")
                    navController.navigate(
                        R.id.productFragment,
                        bundleOf("productId" to id)
                    )
                }
                // Add more routes
            }
        }
    }
}
```

### With Jetpack Compose Navigation

```kotlin
@Composable
fun AppNavigation(startDestination: String = "home") {
    val navController = rememberNavController()

    LaunchedEffect(Unit) {
        Clippr.getInitialLink()?.let { link ->
            navController.navigateToDeepLink(link.path)
        }

        Clippr.onLink = { link ->
            navController.navigateToDeepLink(link.path)
        }
    }

    NavHost(navController, startDestination) {
        composable("home") { HomeScreen() }
        composable("product/{id}") { backStackEntry ->
            ProductScreen(backStackEntry.arguments?.getString("id"))
        }
    }
}

fun NavController.navigateToDeepLink(path: String) {
    when {
        path.startsWith("/product/") -> {
            val id = path.substringAfterLast("/")
            navigate("product/$id")
        }
    }
}
```

## Handling Match Types

```kotlin
fun handleDeepLink(link: ClipprLink) {
    when (link.matchType) {
        MatchType.DIRECT -> {
            // User clicked link with app installed
            navigateAndTrack(link)
        }
        MatchType.DETERMINISTIC -> {
            // 100% accurate - Install Referrer match
            navigateAndTrack(link)
        }
        MatchType.PROBABILISTIC -> {
            val confidence = link.confidence ?: 0.0
            when {
                confidence > 0.8 -> navigateAndTrack(link)
                confidence > 0.5 -> showConfirmation(link)
                else -> handleLowConfidence(link)
            }
        }
        MatchType.NONE -> {
            // Organic install
        }
    }
}
```

## Delayed Navigation

Wait for app initialization:

```kotlin
class AppState {
    var isReady = false
        private set
    private var pendingLink: ClipprLink? = null

    suspend fun initialize() {
        // Load data, authenticate, etc.
        loadUserData()
        isReady = true

        pendingLink?.let {
            handleDeepLink(it)
            pendingLink = null
        }
    }

    fun receivedLink(link: ClipprLink) {
        if (isReady) {
            handleDeepLink(link)
        } else {
            pendingLink = link
        }
    }
}
```

## Error Handling

```kotlin
lifecycleScope.launch {
    try {
        val link = Clippr.getInitialLink()
        link?.let { handleDeepLink(it) }
    } catch (e: Exception) {
        Log.e("Clippr", "Error getting link", e)
        // Continue without link
    }
}

fun handleDeepLink(link: ClipprLink) {
    try {
        navigateToPath(link.path)
    } catch (e: Exception) {
        Log.e("Clippr", "Navigation failed", e)
        navigateToHome()
    }
}
```

## Next Steps

- [Creating Links](/sdks/android/creating-links) - Generate short links
- [Event Tracking](/sdks/android/event-tracking) - Track conversions
- [API Reference](/sdks/android/api-reference) - Complete API docs

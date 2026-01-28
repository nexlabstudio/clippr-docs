---
title: Quick Start
description: Get Clippr integrated in your app in under 5 minutes
---

# Quick Start

This guide will help you create your first deep link and integrate Clippr into your mobile app.

## Prerequisites

- A Clippr account ([sign up here](https://app.useclippr.xyz))
- A mobile app (Flutter, iOS, or Android)

## Step 1: Create Your App in the Dashboard

<Steps>

### Log in to the Clippr Dashboard
Go to [app.useclippr.xyz](https://app.useclippr.xyz) and sign in.

### Create a new organization
If this is your first time, create an organization to manage your apps and team.

### Create a new app
Click "Create App" and enter:
- **App Name**: Your app's display name
- **Subdomain**: Choose a unique subdomain (e.g., `myapp` → `myapp.clppr.xyz`)

### Configure platform settings
Add your platform-specific details:

**For iOS:**
- Bundle ID (e.g., `com.yourcompany.yourapp`)
- App Store ID (optional, for store redirects)
- Team ID (found in Apple Developer Portal)

**For Android:**
- Package Name (e.g., `com.yourcompany.yourapp`)
- SHA256 Fingerprints (for App Links verification)

### Copy your API Key
Save the API key shown after creating your app. You'll need this for SDK initialization.

</Steps>

## Step 2: Install the SDK

Choose your platform:

<Tabs>
<Tab name="Flutter">

Add to your `pubspec.yaml`:

```yaml
dependencies:
  clippr: ^0.0.4
```

Then run:

```bash
flutter pub get
```

</Tab>
<Tab name="iOS">

**Swift Package Manager (Recommended)**

In Xcode: File → Add Packages → Enter:
```
https://github.com/nexlabstudio/clippr-ios.git
```

**CocoaPods**

```ruby
pod 'ClipprSDK', '~> 0.0.4'
```

</Tab>
<Tab name="Android">

Add to your `build.gradle.kts`:

```kotlin
dependencies {
    implementation("xyz.useclippr:clippr:0.0.4")
}
```

</Tab>
</Tabs>

## Step 3: Initialize the SDK

<Tabs>
<Tab name="Flutter">

```dart
import 'package:clippr/clippr.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Clippr.initialize(apiKey: 'YOUR_API_KEY');

  runApp(MyApp());
}
```

</Tab>
<Tab name="iOS">

```swift
import ClipprSDK

@main
struct MyApp: App {
    init() {
        Clippr.initialize(apiKey: "YOUR_API_KEY")
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

</Tab>
<Tab name="Android">

```kotlin
import xyz.useclippr.sdk.Clippr

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        Clippr.initialize(context = this, apiKey = "YOUR_API_KEY")
    }
}
```

</Tab>
</Tabs>

## Step 4: Handle Deep Links

<Tabs>
<Tab name="Flutter">

```dart
class _MyAppState extends State<MyApp> {
  @override
  void initState() {
    super.initState();
    _initDeepLinks();
  }

  Future<void> _initDeepLinks() async {
    // Get link that opened the app (works for deferred links too!)
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
    print('Deep link path: ${link.path}');
    // Navigate based on path
  }
}
```

</Tab>
<Tab name="iOS">

```swift
struct ContentView: View {
    var body: some View {
        NavigationStack {
            // Your content
        }
        .task {
            // Get the link that opened the app
            if let link = await Clippr.getInitialLink() {
                handleDeepLink(link)
            }

            // Listen for links while app is running
            Clippr.onLink = { link in
                handleDeepLink(link)
            }
        }
    }

    func handleDeepLink(_ link: ClipprLink) {
        print("Deep link path: \(link.path)")
        // Navigate based on path
    }
}
```

</Tab>
<Tab name="Android">

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Handle App Link that opened this activity
        Clippr.handle(intent)

        lifecycleScope.launch {
            // Get the link that opened the app
            Clippr.getInitialLink()?.let { link ->
                handleDeepLink(link)
            }
        }

        // Listen for links while app is running
        Clippr.onLink = { link ->
            handleDeepLink(link)
        }
    }

    private fun handleDeepLink(link: ClipprLink) {
        Log.d("Clippr", "Deep link path: ${link.path}")
        // Navigate based on path
    }
}
```

</Tab>
</Tabs>

## Step 5: Configure Platform Deep Linking

<Tabs>
<Tab name="iOS">

Add Associated Domains in Xcode:

1. Select your target → Signing & Capabilities
2. Click "+ Capability" and add "Associated Domains"
3. Add: `applinks:yourapp.clppr.xyz`

That's it! Clippr automatically hosts your AASA file.

</Tab>
<Tab name="Android">

Add intent filter to `AndroidManifest.xml`:

```xml
<activity android:name=".MainActivity">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />

        <data
            android:scheme="https"
            android:host="yourapp.clppr.xyz" />
    </intent-filter>
</activity>
```

Clippr automatically hosts your Asset Links file.

</Tab>
</Tabs>

## Step 6: Create Your First Link

Go back to the Clippr Dashboard:

1. Navigate to **Links** → **Create Link**
2. Enter a **Deep Link Path** (e.g., `/product/123`)
3. Optionally add:
   - Custom alias (e.g., `summer-sale`)
   - Campaign, source, medium for attribution
   - Social preview metadata
4. Click **Create**

Your link is ready: `https://yourapp.clppr.xyz/summer-sale`

## Step 7: Test It!

1. Send the link to yourself
2. Click it on your test device:
   - **With app installed**: App opens to your deep link path
   - **Without app installed**: Redirected to app store, then app opens to the path after install

<Info>
Enable debug mode during development to see detailed logs:

```dart
await Clippr.initialize(apiKey: 'YOUR_API_KEY', debug: true);
```
</Info>

## Next Steps

- [Concepts](/getting-started/concepts) - Understand how deep linking and attribution work
- [Flutter SDK](/sdks/flutter) - Detailed Flutter integration guide
- [iOS SDK](/sdks/ios) - Detailed iOS integration guide
- [Android SDK](/sdks/android) - Detailed Android integration guide
- [Dashboard Guide](/dashboard) - Learn to use all dashboard features

---
title: SKAdNetwork Integration
description: Privacy-preserving attribution with Apple's SKAdNetwork
---

# SKAdNetwork Integration

SKAdNetwork (SKAN) is Apple's privacy-preserving attribution framework for iOS. Clippr supports SKAN for measuring ad campaign effectiveness without user-level tracking.

## What is SKAdNetwork?

SKAdNetwork allows advertisers to measure conversions while preserving user privacy:

- No user identifiers shared
- Aggregated, anonymized data
- Apple-verified install attribution
- Delayed postbacks (24-48 hours)

## How It Works

```
1. User sees ad in an app (source app)
2. User clicks ad → Installs your app (advertised app)
3. App notifies SKAdNetwork of conversion value
4. Apple sends postback to ad network (after delay)
5. Clippr receives and stores postback
```

## Clippr SKAN Features

- **Postback reception**: Receive and store SKAN postbacks
- **Conversion value schema**: Configure 6-bit conversion values
- **Coarse conversion values**: Support for iOS 16.1+ coarse values
- **Dashboard analytics**: View SKAN data alongside other metrics

## Setup

### 1. Register Your App

Your app must be registered with Apple and have a valid App Store ID configured in Clippr.

### 2. Configure Info.plist

Add the SKAdNetwork identifiers for your ad networks:

```xml
<key>SKAdNetworkItems</key>
<array>
    <dict>
        <key>SKAdNetworkIdentifier</key>
        <string>cstr6suwn9.skadnetwork</string>
    </dict>
    <!-- Add more ad network IDs -->
</array>
```

### 3. Update Conversion Values

When users complete valuable actions, update the conversion value:

```swift
import StoreKit

// User completed sign up (value = 1)
SKAdNetwork.updatePostbackConversionValue(1) { error in
    if let error = error {
        print("Failed to update conversion value: \(error)")
    }
}

// User made purchase (value = 5)
SKAdNetwork.updatePostbackConversionValue(5) { error in
    // Handle error
}
```

## Conversion Value Schema

Configure what each conversion value means in the dashboard:

### 6-bit Values (0-63)

| Value | Meaning |
|-------|---------|
| 0 | Install only |
| 1 | Sign up |
| 2 | Tutorial complete |
| 3 | First purchase |
| 4-10 | Revenue tiers |
| 11-20 | Engagement levels |
| ... | Custom definitions |

### Coarse Values (iOS 16.1+)

| Value | Meaning |
|-------|---------|
| low | Low value user |
| medium | Medium value user |
| high | High value user |

## Viewing SKAN Data

In the dashboard:

1. Go to your app
2. Click **SKAdNetwork**
3. View:
   - Postback volume
   - Conversion value distribution
   - Redownload vs. new install ratio

## Postback Data

Each SKAN postback contains:

| Field | Description |
|-------|-------------|
| `transaction_id` | Unique identifier |
| `app_id` | Your App Store ID |
| `conversion_value` | 0-63 value you set |
| `coarse_conversion_value` | low/medium/high |
| `did_win` | Whether this was the winning ad |
| `fidelity_type` | View-through vs. click-through |
| `redownload` | New install or reinstall |

## Best Practices

### 1. Plan Your Conversion Value Schema

Map business events to values strategically:

```
0: Install
1-10: Engagement milestones
11-30: Revenue tiers ($1-$10, $10-$50, etc.)
31-63: Custom high-value events
```

### 2. Update Values Incrementally

Only update to higher values:

```swift
// Good - values increase over time
updatePostbackConversionValue(1) // Sign up
updatePostbackConversionValue(5) // First purchase
updatePostbackConversionValue(10) // $50 revenue

// Bad - decreasing values are ignored
updatePostbackConversionValue(10)
updatePostbackConversionValue(5) // This is ignored
```

### 3. Consider Timer Windows

SKAN has timer windows that affect when postbacks are sent:
- First 24 hours: Can update value
- After 24 hours: Value is locked
- iOS 16.1+: Multiple postback windows

### 4. Test in Development

Use the SKAdNetwork testing tools:
- Xcode's StoreKit Configuration
- TestFlight for real postbacks
- Apple's SKAN testing profile

## Limitations

- **Delayed data**: Postbacks arrive 24-48+ hours after install
- **Limited values**: Only 64 possible conversion values
- **No user-level data**: Cannot link to specific users
- **Crowd anonymity**: Low-volume campaigns may not receive postbacks

## SKAN 4.0 Features

iOS 16.1+ supports SKAN 4.0:

- **Multiple postbacks**: Up to 3 postbacks over time
- **Coarse values**: Low/medium/high for smaller campaigns
- **Source identifier**: 4-digit campaign identifier
- **Lock window**: Extended conversion windows

## Troubleshooting

### Not Receiving Postbacks

1. Verify App Store ID is correct
2. Check SKAdNetwork identifiers in Info.plist
3. Ensure you're calling `updatePostbackConversionValue`
4. Wait 24-48 hours (postbacks are delayed)

### Conversion Values Not Updating

1. Can only increase values, not decrease
2. Window may have expired (24+ hours)
3. Check for errors in the callback

## Next Steps

- [iOS SDK](/sdks/ios) - Implement the iOS SDK
- [Analytics](/dashboard/analytics) - View SKAN data in dashboard

---
title: Changelog
description: Release notes and version history
---

# Changelog

Release notes for Clippr SDKs and platform updates.

## SDKs

### Flutter SDK

#### v0.0.4 (Current)

- Initial public release
- Deep link handling
- Deferred deep linking support
- Event and revenue tracking
- Short link creation
- Platform channel implementation for iOS and Android

#### Roadmap

- Improved error handling
- Offline event queuing
- React Native support

---

### iOS SDK

#### v0.0.4 (Current)

- Initial public release
- Universal Link handling
- Deferred deep linking with fingerprinting
- Async/await and completion handler APIs
- Event and revenue tracking
- Short link creation
- ATT integration for IDFA

#### Roadmap

- SwiftUI deep link modifier
- Improved caching
- Offline support

---

### Android SDK

#### v0.0.4 (Current)

- Initial public release
- App Link handling
- Install Referrer integration (100% accurate attribution)
- Deferred deep linking with fingerprinting
- Kotlin coroutines and Java callback APIs
- Event and revenue tracking
- Short link creation

#### Roadmap

- Jetpack Compose integration
- WorkManager for offline events
- Improved retry logic

---

## Platform

### January 2025

**Dashboard**
- New analytics dashboard with improved charts
- Team member invitation flow
- Billing portal integration

**API**
- Rate limiting improvements
- Better error messages
- OpenAPI documentation

**Infrastructure**
- Edge worker improvements for faster link resolution
- Database query optimizations

### December 2024

**Initial Launch**
- Core deep linking functionality
- Dashboard for link management
- Flutter, iOS, and Android SDKs
- Basic analytics
- Organization and team management

---

## API Versions

### v1 (Current)

The current stable API version. All endpoints are prefixed with `/v1`.

No breaking changes planned. New features will be added as new endpoints.

---

## Deprecations

No deprecations at this time.

---

<!-- Migration Guides section disabled — Clippr positioning is attribution-first, not FDL replacement.
## Migration Guides

- [Firebase Dynamic Links Migration](/guides/migration) - Migrate from Firebase before August 2025

---
-->

## Reporting Issues

Found a bug or have a feature request?

- **Flutter SDK**: [GitHub Issues](https://github.com/nexlabstudio/clippr-flutter/issues)
- **iOS SDK**: [GitHub Issues](https://github.com/nexlabstudio/clippr-ios-sdk/issues)
- **Android SDK**: [GitHub Issues](https://github.com/nexlabstudio/clippr-android-sdk/issues)
- **Dashboard/API**: Email support@useclippr.xyz

---
title: Attribution Guide
description: Understanding and using attribution data in Clippr
---

# Attribution Guide

Attribution helps you understand where your users come from and which marketing channels drive the best results.

## What is Attribution?

Attribution connects user actions (installs, purchases, sign-ups) back to their original source:

```
User sees Instagram ad → Clicks link → Installs app → Makes purchase
                         ↓
Attribution tells you: This $99 purchase came from Instagram summer campaign
```

## Attribution Parameters

When creating links, add these parameters:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `campaign` | Marketing campaign name | `summer_sale_2024` |
| `source` | Traffic source | `instagram`, `google`, `email` |
| `medium` | Marketing medium | `social`, `cpc`, `newsletter` |

### Creating Attributed Links

**In the Dashboard:**

1. Create a new link
2. Fill in Campaign, Source, Medium fields

**Via SDK:**

```dart
final params = LinkParameters(
  path: '/product/123',
  campaign: 'summer_sale',
  source: 'instagram',
  medium: 'story',
);
final link = await Clippr.createLink(params);
```

## Reading Attribution Data

When a user opens your app via a link:

```dart
final link = await Clippr.getInitialLink();
if (link != null) {
  final attribution = link.attribution;
  if (attribution != null) {
    print('Campaign: ${attribution.campaign}');
    print('Source: ${attribution.source}');
    print('Medium: ${attribution.medium}');
  }
}
```

## Common Attribution Patterns

### Social Media Campaigns

```dart
// Instagram story ad
LinkParameters(
  path: '/sale',
  campaign: 'summer_sale_2024',
  source: 'instagram',
  medium: 'story',
)

// Facebook feed ad
LinkParameters(
  path: '/sale',
  campaign: 'summer_sale_2024',
  source: 'facebook',
  medium: 'feed',
)

// TikTok video
LinkParameters(
  path: '/sale',
  campaign: 'summer_sale_2024',
  source: 'tiktok',
  medium: 'organic',
)
```

### Email Marketing

```dart
LinkParameters(
  path: '/offer',
  campaign: 'weekly_newsletter',
  source: 'email',
  medium: 'newsletter',
)

LinkParameters(
  path: '/cart',
  campaign: 'abandoned_cart',
  source: 'email',
  medium: 'automation',
)
```

### Paid Advertising

```dart
// Google Ads
LinkParameters(
  path: '/landing',
  campaign: 'brand_awareness_q1',
  source: 'google',
  medium: 'cpc',
)

// Apple Search Ads
LinkParameters(
  path: '/app',
  campaign: 'app_install',
  source: 'apple',
  medium: 'search_ads',
)
```

### Referral Programs

```dart
LinkParameters(
  path: '/signup',
  campaign: 'referral_program',
  source: 'user_invite',
  medium: 'referral',
  metadata: {
    'referrer_id': userId,
  },
)
```

### Influencer Marketing

```dart
LinkParameters(
  path: '/product/xyz',
  campaign: 'influencer_q2',
  source: 'youtube',
  medium: 'creator_${creatorId}',
)
```

## Attribution in Analytics

The Clippr dashboard shows:

### By Campaign

| Campaign | Clicks | Installs | Revenue |
|----------|--------|----------|---------|
| summer_sale_2024 | 10,000 | 2,500 | $45,000 |
| referral_program | 5,000 | 1,200 | $28,000 |
| brand_awareness | 25,000 | 3,000 | $12,000 |

### By Source

| Source | Clicks | Installs | Cost/Install |
|--------|--------|----------|--------------|
| instagram | 15,000 | 4,000 | $2.50 |
| google | 10,000 | 2,000 | $5.00 |
| email | 8,000 | 1,500 | $0.00 |

### By Medium

| Medium | Clicks | Conversion Rate |
|--------|--------|-----------------|
| social | 20,000 | 25% |
| cpc | 15,000 | 18% |
| email | 8,000 | 35% |

## Event Attribution

Track events and attribute them to the original link:

```dart
// User came from Instagram summer sale campaign
// Now they're making a purchase

await Clippr.trackRevenue(
  'purchase',
  revenue: 99.99,
  currency: 'USD',
  params: {
    'order_id': orderId,
  },
);

// In analytics, this $99.99 is attributed to:
// Campaign: summer_sale_2024
// Source: instagram
// Medium: story
```

## Multi-Touch Attribution

By default, Clippr uses **last-touch attribution**:

```
Click 1 (Google) → Click 2 (Instagram) → Install → Purchase
                                           ↑
                              Attributed to Instagram
```

The last click before install gets credit.

## Best Practices

### Be Consistent

Use the same naming conventions across all campaigns:

```
Good:
- campaign: summer_sale_2024
- campaign: black_friday_2024

Bad:
- campaign: Summer Sale
- campaign: bf24
- campaign: BlackFriday
```

### Be Specific

More specific attribution = better insights:

```
// Too vague
source: 'social'

// Better
source: 'instagram'

// Best (if you have multiple campaigns)
source: 'instagram'
medium: 'story'
campaign: 'summer_shoes_june'
```

### Track Everything

Create unique links for each placement:

- One link per ad creative
- One link per email
- One link per influencer
- One link per offline material (with QR codes)

### Use Metadata for Custom Data

Attribution parameters are limited. Use metadata for additional tracking:

```dart
LinkParameters(
  path: '/product/123',
  campaign: 'influencer_q2',
  source: 'youtube',
  medium: 'description_link',
  metadata: {
    'influencer_id': 'creator_456',
    'video_id': 'abc123',
    'placement': 'top_of_description',
  },
)
```

## Next Steps

- [Dashboard Analytics](/dashboard/analytics) - View attribution reports
- [Event Tracking](/sdks/flutter/event-tracking) - Track conversions

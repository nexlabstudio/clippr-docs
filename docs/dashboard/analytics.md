---
title: Analytics
description: Track clicks, installs, and conversions in the Clippr dashboard
---

# Analytics

The Clippr dashboard provides comprehensive analytics to track your deep link performance.

## Overview Metrics

The dashboard home shows key metrics:

| Metric | Description |
|--------|-------------|
| **Total Clicks** | All link clicks in the selected period |
| **Installs** | App installs attributed to links |
| **Match Rate** | Percentage of clicks that resulted in matched installs |
| **Active Links** | Links that received clicks |

## Date Range

Select a time period to analyze:

- Last 7 days
- Last 30 days
- Last 90 days
- This month
- Last month
- Custom range

## Clicks Over Time

The line chart shows click trends:

- View by hour, day, week, or month
- Identify traffic patterns
- Spot campaign launch impacts

## Top Links

See your best-performing links:

| Column | Description |
|--------|-------------|
| **Link** | The short URL or alias |
| **Clicks** | Total clicks |
| **Installs** | Attributed installs |
| **Match Rate** | Click-to-install ratio |

## Top Campaigns

Analyze campaign performance:

| Column | Description |
|--------|-------------|
| **Campaign** | Campaign name |
| **Clicks** | Total clicks |
| **Installs** | Attributed installs |
| **Revenue** | Tracked revenue (if applicable) |

## Link-Level Analytics

Click on any link to see detailed stats:

### Click Breakdown

- Total clicks
- Unique clicks (by device)
- Platform split (iOS vs Android)

### Attribution

- Installs attributed
- Match types used (direct, deterministic, probabilistic)
- Confidence distribution

### Geography

- Clicks by country
- Top regions

### Devices

- Platform breakdown
- Device models
- OS versions

## Event Analytics

If you're tracking events via the SDK:

- Event counts by type
- Revenue by event
- Conversion funnel (click → install → event)

## Exporting Data

Export analytics data:

1. Go to **Analytics**
2. Set your date range and filters
3. Click **Export**
4. Choose CSV or JSON format

## Understanding Match Rate

Match rate = (Attributed Installs / Total Clicks) × 100

**Typical ranges:**
- 60-80%: Good performance
- 80-95%: Excellent (likely includes direct links)
- Below 60%: May indicate tracking issues

**Factors affecting match rate:**
- Time between click and install
- User's privacy settings
- Device fingerprint accuracy

## Best Practices

### Regular Monitoring

- Check dashboard weekly
- Set up alerts for unusual patterns
- Track campaign ROI

### Optimize Low Performers

- A/B test link copy and images
- Review targeting for low match rate campaigns
- Update stale link content

### Attribution Windows

Deferred links are matched within:
- 24 hours: High confidence
- 72 hours: Standard window
- Beyond: Lower confidence matches

## Next Steps

- [Team Management](/dashboard/team) - Share access with your team
- [Billing](/dashboard/billing) - Manage your subscription

---
title: Creating Links
description: Create and manage deep links in the Clippr dashboard
---

# Creating Links

Create deep links that open your app directly to specific content.

## Create a Link

1. Navigate to **Links** in the sidebar
2. Click **Create Link**
3. Fill in the details

### Basic Settings

| Field | Description | Example |
|-------|-------------|---------|
| **App** | Select your app | My Awesome App |
| **Deep Link Path** | Path within your app | `/product/123` |
| **Alias** (optional) | Custom short code | `summer-sale` |

### Attribution (Optional)

Track where users come from:

| Field | Description | Example |
|-------|-------------|---------|
| **Campaign** | Marketing campaign | `summer_sale_2024` |
| **Source** | Traffic source | `facebook`, `email` |
| **Medium** | Marketing medium | `social`, `cpc` |

### Fallback URLs (Optional)

Control where users go if they can't open the app:

| Field | Description |
|-------|-------------|
| **iOS Fallback** | URL for iOS users without app |
| **Android Fallback** | URL for Android users without app |
| **Web Fallback** | URL for desktop/web users |

If not specified, users are redirected to the appropriate app store.

### Social Preview (Optional)

Control how your link appears when shared on social media:

| Field | Description |
|-------|-------------|
| **Title** | Preview title (og:title) |
| **Description** | Preview description (og:description) |
| **Image URL** | Preview image (og:image) |

### Custom Metadata (Optional)

Add custom data that's passed to your app:

```json
{
  "discount": "20%",
  "referrer": "user123",
  "expires": "2024-12-31"
}
```

## Link URL

After creating, your link URL is:

```
https://yourapp.clppr.xyz/{alias-or-shortcode}
```

Examples:
- `https://myapp.clppr.xyz/summer-sale` (with alias)
- `https://myapp.clppr.xyz/abc123` (auto-generated)

## Manage Links

### View Links

1. Go to **Links**
2. Browse or search your links
3. Filter by app if needed

### View Link Details

Click on a link to see:
- Full configuration
- QR code
- Click statistics
- Quick copy button

### Edit a Link

1. Click on a link
2. Click **Edit**
3. Update fields
4. Click **Save**

### Delete a Link

1. Click on a link
2. Click **Delete**
3. Confirm deletion

<Warning>
Deleted links return 404 immediately. Consider deactivating instead if you might need it later.
</Warning>

## Bulk Import

Import multiple links from CSV:

1. Go to **Links** → **Import**
2. Upload your CSV file
3. Map columns to fields
4. Review and import

### CSV Format

```csv
path,alias,campaign,source,medium
/product/1,shoe-deal,summer,facebook,social
/product/2,hat-promo,summer,instagram,social
```

## QR Codes

Generate QR codes for any link:

1. Click on a link
2. Click the **QR Code** button
3. Download PNG or SVG

QR codes are useful for:
- Print materials
- In-store displays
- Event marketing

## Link Analytics

Each link tracks:
- Total clicks
- Unique clicks
- Installs attributed
- Match rate

See [Analytics](/dashboard/analytics) for detailed reporting.

## Best Practices

### Use Descriptive Aliases

```
Good:  summer-sale-2024, product-shoes-blue
Bad:   a1b2c3, link1
```

### Always Add Attribution

Track your marketing efforts:
```
campaign: summer_sale
source: facebook
medium: paid_social
```

### Set Social Tags for Shared Links

Make shared links more clickable:
- Compelling title (under 60 chars)
- Clear description (under 155 chars)
- Eye-catching image (1200x630px)

## Next Steps

- [Analytics](/dashboard/analytics) - Track link performance
- [Team Management](/dashboard/team) - Collaborate with your team

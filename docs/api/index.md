---
title: API Overview
description: Clippr REST API documentation
---

# API Overview

The Clippr API allows you to programmatically manage links, track events, and access analytics.

## Base URL

```
https://api.clppr.xyz/v1
```

## Authentication

The API supports two authentication methods:

### Bearer Token (Dashboard Users)

For dashboard users managing links and apps:

```bash
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  https://api.clppr.xyz/v1/links
```

### API Key (SDK/Server)

For server-side integrations and SDK operations:

```bash
curl -H "X-API-Key: YOUR_APP_API_KEY" \
  https://api.clppr.xyz/v1/sdk/links
```

See [Authentication](/api/authentication) for details.

## Endpoints

### Links

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/links` | List all links |
| `POST` | `/links` | Create a link |
| `GET` | `/links/{id}` | Get a link |
| `PATCH` | `/links/{id}` | Update a link |
| `DELETE` | `/links/{id}` | Delete a link |
| `GET` | `/links/{id}/stats` | Get link statistics |

### Apps

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/apps` | List all apps |
| `POST` | `/apps` | Create an app |
| `GET` | `/apps/{id}` | Get an app |
| `PATCH` | `/apps/{id}` | Update an app |
| `DELETE` | `/apps/{id}` | Delete an app |
| `POST` | `/apps/{id}/rotate-key` | Rotate API key |

### Analytics

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/analytics/overview` | Get aggregate stats |
| `GET` | `/analytics/timeseries` | Get time-series data |
| `GET` | `/analytics/top-links` | Get top performing links |
| `GET` | `/analytics/top-campaigns` | Get top campaigns |

### SDK Endpoints

These use API Key authentication:

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/sdk/match` | Match device to click |
| `POST` | `/sdk/install` | Track installation |
| `POST` | `/sdk/events` | Track events |
| `POST` | `/sdk/links` | Create link from SDK |

## Request Format

All requests should use JSON:

```bash
curl -X POST https://api.clppr.xyz/v1/links \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "app_id": "app_123",
    "path": "/product/456",
    "campaign": "summer_sale"
  }'
```

## Response Format

Responses are JSON:

```json
{
  "id": "link_789",
  "short_code": "abc123",
  "url": "https://myapp.clppr.xyz/abc123",
  "path": "/product/456",
  "created_at": "2024-01-15T10:30:00Z"
}
```

## Errors

Errors include a code and message:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Path is required",
    "details": {
      "field": "path"
    }
  }
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Invalid or missing authentication |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 400 | Invalid request data |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server error |

## Rate Limiting

| Endpoint Type | Limit |
|---------------|-------|
| Authentication | 5/min per IP |
| Link creation | 30/min per user |
| SDK match | 1000/min per app |
| SDK events | 500/min per app |
| General | 100/min per IP |

Rate limit headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1642234567
```

## Pagination

List endpoints support pagination:

```bash
GET /v1/links?limit=20&offset=0
```

Response includes pagination info:

```json
{
  "data": [...],
  "pagination": {
    "total": 150,
    "limit": 20,
    "offset": 0,
    "has_more": true
  }
}
```

## Next Steps

- [Authentication](/api/authentication) - Detailed auth guide
- [SDK Integration](/sdks) - Use the official SDKs

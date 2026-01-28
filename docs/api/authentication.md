---
title: Authentication
description: Authenticate with the Clippr API
---

# Authentication

The Clippr API supports multiple authentication methods depending on your use case.

## Authentication Methods

| Method | Use Case | Header |
|--------|----------|--------|
| Bearer Token | Dashboard integrations | `Authorization: Bearer {token}` |
| API Key | SDK/Server integrations | `X-API-Key: {key}` |

## Bearer Token Authentication

For users accessing the API on behalf of their account.

### Obtaining a Token

Tokens are obtained via the authentication endpoints:

```bash
POST /v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "your_password"
}
```

Response:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_in": 900
}
```

### Using the Token

Include the token in the Authorization header:

```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  https://api.clppr.xyz/v1/links
```

### Token Expiration

- Access tokens expire in 15 minutes
- Refresh tokens expire in 7 days

### Refreshing Tokens

```bash
POST /v1/auth/refresh
Content-Type: application/json

{
  "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
}
```

## API Key Authentication

For SDK and server-side integrations.

### Obtaining an API Key

API keys are generated when you create an app in the dashboard:

1. Go to **Apps** → **Create App**
2. Copy the API key shown after creation

Or for existing apps:
1. Go to **Apps** → Select your app
2. View the API key in the app details

### Using the API Key

Include the key in the `X-API-Key` header:

```bash
curl -H "X-API-Key: your_app_api_key" \
  https://api.clppr.xyz/v1/sdk/links
```

Or in the Authorization header:

```bash
curl -H "Authorization: Bearer your_app_api_key" \
  https://api.clppr.xyz/v1/sdk/links
```

### API Key Permissions

API keys can only access:
- `/v1/sdk/*` endpoints
- Resources belonging to that app

### Rotating API Keys

If your key is compromised:

1. Go to **Apps** → Select your app
2. Click **Rotate API Key**
3. Update your app and release an update

<Warning>
Rotating immediately invalidates the old key.
</Warning>

## OAuth (Optional)

The API also supports OAuth login via Google and GitHub if configured.

### OAuth Flow

1. Start OAuth:
   ```
   POST /v1/auth/oauth/{provider}/start
   ```

2. User authorizes in browser

3. Handle callback:
   ```
   POST /v1/auth/oauth/{provider}/callback
   {
     "code": "authorization_code",
     "state": "state_parameter"
   }
   ```

4. Receive tokens in response

## Security Best Practices

### Protect Your Credentials

- Never commit API keys to version control
- Use environment variables
- Rotate keys periodically

### Use HTTPS

All API requests must use HTTPS. HTTP requests are rejected.

### Implement Token Refresh

Don't store refresh tokens in client-side code. Handle token refresh server-side when possible.

### Rate Limit Handling

Implement exponential backoff when you receive 429 errors:

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    const response = await fetch(url, options);

    if (response.status === 429) {
      const retryAfter = response.headers.get('Retry-After') || Math.pow(2, i);
      await sleep(retryAfter * 1000);
      continue;
    }

    return response;
  }
  throw new Error('Max retries exceeded');
}
```

## Error Responses

### 401 Unauthorized

```json
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Invalid or expired token"
  }
}
```

**Causes:**
- Missing authentication header
- Invalid token/key
- Expired access token

### 403 Forbidden

```json
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Insufficient permissions"
  }
}
```

**Causes:**
- API key accessing non-SDK endpoints
- User lacking organization access
- Resource belongs to different organization

## Next Steps

- [API Overview](/api) - Full endpoint reference
- [SDK Documentation](/sdks) - Use official SDKs

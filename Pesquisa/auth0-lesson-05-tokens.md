---
title: "Auth0 Lesson 5 — Tokens"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - status/rascunho
area: authentication
---

# 🎟️ Auth0 Lesson 5 — Tokens

## 🎯 Context

Tokens are the artifacts that carry identity and authorization information between parties. Understanding the purpose, format, and lifecycle of each token type is essential for building secure applications with Auth0.

## 📊 Token Overview

| Token | Purpose | Format | Audience | Who Creates It | Who Consumes It |
|-------|---------|--------|----------|----------------|-----------------|
| **Access Token** | Authorize API access | Opaque or JWT | Resource Server (API) | Auth0 | Your API |
| **ID Token** | Identify the user | JWT (always) | Client Application | Auth0 | Your App |
| **Refresh Token** | Get new access tokens | Opaque string | Auth0 | Auth0 | Your App |

## 🔑 Access Token

### Purpose

The Access Token is a credential that authorizes the bearer to access a protected resource (your API). It tells the API: "the bearer of this token has been granted permission to perform these actions."

### Opaque vs. JWT Format

Auth0 issues access tokens in two formats depending on the configuration:

#### Opaque Access Token

```
kB2VyZTI4NjM4OTBhMmE4
```

- A random, opaque string with no readable information
- To validate: the API must call Auth0's `/userinfo` or token introspection endpoint
- **When issued**: When no `audience` parameter is specified, or when the audience is Auth0's `/userinfo` endpoint
- Requires an online call to Auth0 for every validation

#### JWT Access Token

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwczovL...
```

- A self-contained JWT with claims
- To validate: verify the signature locally using Auth0's public key (from JWKS)
- **When issued**: When an `audience` parameter is specified that matches a registered API
- Can be validated **offline** (no call to Auth0 required)

> [!important] Getting a JWT Access Token
> You **must** specify the `audience` parameter (matching a registered API identifier) in your authorization request to receive a JWT access token. Without it, you'll get an opaque token.

```javascript
// This returns an OPAQUE token (no audience specified)
const auth0 = await createAuth0Client({
  domain: 'my-tenant.auth0.com',
  clientId: 'abc123'
  // No audience → opaque access token
});

// This returns a JWT token (audience specified)
const auth0 = await createAuth0Client({
  domain: 'my-tenant.auth0.com',
  clientId: 'abc123',
  authorizationParams: {
    audience: 'https://api.myapp.com'  // ← This triggers JWT format
  }
});
```

### Access Token Anatomy (JWT format)

```json
{
  "iss": "https://my-tenant.auth0.com/",
  "sub": "auth0|507f1f77bcf86cd799439011",
  "aud": [
    "https://api.myapp.com",
    "https://my-tenant.auth0.com/userinfo"
  ],
  "iat": 1681321200,
  "exp": 1681407600,
  "azp": "abc123clientid",
  "scope": "openid profile email read:data write:data",
  "permissions": ["read:data", "write:data"],
  "gty": "authorization_code"
}
```

| Claim | Meaning |
|-------|---------|
| `iss` | Issuer — your Auth0 tenant URL |
| `sub` | Subject — the user ID |
| `aud` | Audience — the API(s) the token is intended for |
| `iat` | Issued at — when the token was created |
| `exp` | Expiration — when the token expires |
| `azp` | Authorized party — the application that requested the token |
| `scope` | Scopes granted to the token |
| `permissions` | Specific permissions (when RBAC is enabled) |
| `gty` | Grant type used to obtain the token |

### Access Token Expiration

- **Default**: 86400 seconds (24 hours)
- **Configurable** per API in Auth0 Dashboard
- **Recommendation**: Keep short (e.g., 1-8 hours) and use refresh tokens to renew
- Shorter lifetimes = less risk if a token is compromised

### Access Token Validation (on your API)

```javascript
const { auth } = require('express-oauth2-jwt-bearer');

// Middleware that validates the access token
const jwtCheck = auth({
  audience: 'https://api.myapp.com',
  issuerBaseURL: 'https://my-tenant.auth0.com/',
  tokenSigningAlg: 'RS256'
});

app.use('/api', jwtCheck);

app.get('/api/data', (req, res) => {
  // req.auth.payload contains the decoded token
  const userId = req.auth.payload.sub;
  const scopes = req.auth.payload.scope.split(' ');

  if (!scopes.includes('read:data')) {
    return res.status(403).json({ error: 'Insufficient scope' });
  }

  res.json({ data: '...' });
});
```

## 🆔 ID Token

### Purpose

The ID Token is a JWT that contains claims about the **authenticated user**. It is meant for the **client application** (your frontend or backend), not for your API.

> [!danger] Critical Rule
> **Never send an ID Token to your API as an authorization credential.** The ID Token is for the client application. The Access Token is for the API. Mixing them up is a common security mistake.

### When to Use ID Token vs. Access Token

| Scenario | Use |
|----------|-----|
| Display user's name/email in the UI | ID Token |
| Call your backend API | Access Token |
| Check if user is authenticated | ID Token |
| Determine what user can access | Access Token (scopes/permissions) |
| Store user info in frontend state | ID Token claims |
| Authorize a microservice call | Access Token |

### ID Token Anatomy

```json
{
  "iss": "https://my-tenant.auth0.com/",
  "sub": "auth0|507f1f77bcf86cd799439011",
  "aud": "abc123clientid",
  "exp": 1681324800,
  "iat": 1681321200,
  "nonce": "random_nonce_value",
  "at_hash": "HK6E_P6Dh8Y93mRNtsDB1Q",
  "name": "Alice Smith",
  "email": "alice@example.com",
  "email_verified": true,
  "picture": "https://example.com/alice.jpg",
  "updated_at": "2026-04-13T00:00:00.000Z"
}
```

| Claim | Source |
|-------|--------|
| `iss`, `sub`, `aud`, `exp`, `iat` | Standard JWT claims |
| `nonce` | Random value sent in the auth request, used to prevent replay attacks |
| `at_hash` | Hash of the access token, for binding ID token to access token |
| `name`, `email`, `picture`, etc. | OIDC standard claims (depend on requested scopes) |

### ID Token Claims by Scope

| Scope Requested | Claims Added to ID Token |
|-----------------|-------------------------|
| `openid` | `sub` |
| `profile` | `name`, `nickname`, `picture`, `updated_at` |
| `email` | `email`, `email_verified` |
| `phone` | `phone_number`, `phone_number_verified` |
| `address` | `address` |

### Adding Custom Claims to ID Tokens

Use Auth0 **Actions** to add custom claims:

```javascript
// Auth0 Action: Post Login
exports.onExecutePostLogin = async (event, api) => {
  const namespace = 'https://myapp.com/claims';

  // Add user metadata to ID token
  api.idToken.setCustomClaim(`${namespace}/language`,
    event.user.user_metadata?.language || 'en'
  );

  // Add app metadata to ID token
  api.idToken.setCustomClaim(`${namespace}/plan`,
    event.user.app_metadata?.plan || 'free'
  );

  // Add roles
  api.idToken.setCustomClaim(`${namespace}/roles`,
    event.authorization?.roles || []
  );
};
```

> [!note] Namespace Requirement
> Custom claims must use a **namespaced** key (a URL you control) to avoid collisions with standard OIDC claims. Auth0 will ignore private non-namespaced custom claims.

## 🔄 Refresh Token

### Purpose

A Refresh Token is a long-lived credential that allows the application to obtain **new access tokens without requiring the user to re-authenticate**. This is critical for:

- Maintaining user sessions across access token expirations
- Avoiding frequent redirects to the login page
- Enabling "remember me" functionality

### How Refresh Tokens Work

```
┌──────────────┐          ┌──────────────┐
│  Your App     │          │  Auth0        │
└──────┬───────┘          └──────┬───────┘
       │                         │
       │ (Access token expired)  │
       │                         │
       │ POST /oauth/token       │
       │ {grant_type:            │
       │  "refresh_token",       │
       │  refresh_token: "abc",  │
       │  client_id: "..."}      │
       │────────────────────────►│
       │                         │
       │ {new_access_token,      │
       │  new_refresh_token,     │
       │  new_id_token}          │
       │◄────────────────────────│
```

```javascript
// Example: Using refresh token to get new access token
const response = await fetch('https://my-tenant.auth0.com/oauth/token', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    grant_type: 'refresh_token',
    client_id: 'YOUR_CLIENT_ID',
    refresh_token: storedRefreshToken
  })
});

const { access_token, refresh_token, id_token } = await response.json();
// Store the new refresh_token (if rotation is enabled, it will be different)
```

### Refresh Token Rotation

Auth0 supports **Refresh Token Rotation**, which is a critical security feature:

- Every time a refresh token is used, a **new refresh token** is issued and the **old one is invalidated**
- If an attacker steals and uses an old refresh token, Auth0 detects the **reuse** and **revokes all tokens** for that session
- **Required for SPAs** (since they can't store secrets securely)

```
Normal flow:
  App uses RT-1 → Gets AT-2 + RT-2 (RT-1 invalidated)
  App uses RT-2 → Gets AT-3 + RT-3 (RT-2 invalidated)

Attack detection:
  Attacker steals RT-1
  App uses RT-2 (normal) → Gets AT-3 + RT-3
  Attacker uses RT-1 (stolen) → Auth0 detects reuse!
  → ALL refresh tokens for this session are revoked
  → User must re-authenticate
```

### Expiration Settings

Auth0 provides two expiration mechanisms:

#### Absolute Expiration

The refresh token expires after a fixed time, regardless of activity:

- **Purpose**: Maximum session lifetime
- **Example**: Token expires 30 days after issuance, even if actively used

#### Inactivity Expiration

The refresh token expires after a period of inactivity (no token refresh requests):

- **Purpose**: Expire idle sessions
- **Example**: Token expires after 7 days of no use

```
Timeline (absolute: 30 days, inactivity: 7 days):

Day 0:  Token issued
Day 3:  Token used → Reset inactivity timer
Day 6:  Token used → Reset inactivity timer
Day 14: Token NOT used...
Day 21: Token expires (7 days of inactivity)

OR:

Day 0:  Token issued
Day 5:  Token used → Reset inactivity timer
Day 10: Token used → Reset inactivity timer
Day 28: Token used → Reset inactivity timer
Day 30: Token expires (absolute expiration reached, regardless of activity)
```

### Refresh Token Limits

- **Maximum 200 active refresh tokens** per user per application
- When the limit is reached, the **oldest token is automatically revoked**
- Expired and revoked tokens do **not** count against the limit

### Refresh Token Security Best Practices

| Practice | Reason |
|----------|--------|
| **Enable rotation** | Prevents stolen tokens from being used indefinitely |
| **Set reasonable lifetimes** | Balance security vs. UX |
| **Store securely** | Use secure storage (Keychain on iOS, EncryptedSharedPreferences on Android) |
| **Never expose in URLs** | Refresh tokens should only travel in POST bodies |
| **Revoke on logout** | Clean up tokens when users explicitly log out |
| **Monitor for reuse** | Auth0 automatically detects refresh token reuse with rotation enabled |

### Where Refresh Tokens Are Available

| Flow | Refresh Token Available? |
|------|------------------------|
| Authorization Code | Yes |
| Authorization Code + PKCE | Yes (with rotation recommended) |
| Client Credentials | No (not needed — just request a new access token) |
| Device Authorization | Yes |
| Implicit | No (deprecated flow) |
| Resource Owner Password | Yes (deprecated flow) |

> [!important] Requesting Refresh Tokens
> To receive a refresh token, you must include `offline_access` in the `scope` parameter of your authorization request. The API must also have "Allow Offline Access" enabled in the Auth0 Dashboard.

```javascript
// Requesting a refresh token
const auth0 = await createAuth0Client({
  domain: 'my-tenant.auth0.com',
  clientId: 'abc123',
  authorizationParams: {
    audience: 'https://api.myapp.com',
    scope: 'openid profile email offline_access'  // ← offline_access for refresh token
  },
  useRefreshTokens: true  // ← Tells the SDK to use refresh tokens
});
```

## 🔄 Token Lifecycle and Management

### Complete Token Lifecycle

```
1. USER AUTHENTICATES
   ├── Auth0 issues: Access Token + ID Token + Refresh Token
   │
2. APP USES ACCESS TOKEN
   ├── Sends in Authorization header to API
   ├── API validates token (signature, expiration, audience, scopes)
   │
3. ACCESS TOKEN EXPIRES
   ├── API returns 401 Unauthorized
   │
4. APP REFRESHES
   ├── Sends Refresh Token to /oauth/token
   ├── Auth0 returns: New Access Token + New ID Token + New Refresh Token*
   │   (* if rotation is enabled)
   │
5. REPEAT steps 2-4
   │
6. SESSION ENDS (one of):
   ├── User logs out → App revokes refresh token
   ├── Inactivity timeout → Refresh token expires
   ├── Absolute timeout → Refresh token expires
   └── Security event → Auth0 revokes all tokens
```

### Silent Authentication (for SPAs)

Before refresh token support was added to SPAs, Auth0 used **silent authentication** via a hidden iframe:

```javascript
// Legacy approach (hidden iframe)
// The SDK checks if there's an active Auth0 session
// by sending a request in a hidden iframe to /authorize with prompt=none
const token = await auth0.getAccessTokenSilently();
```

Modern approach uses **refresh tokens with rotation**:

```javascript
// Modern approach (refresh tokens)
const auth0 = await createAuth0Client({
  domain: 'my-tenant.auth0.com',
  clientId: 'abc123',
  useRefreshTokens: true,       // Use refresh tokens instead of iframe
  cacheLocation: 'localstorage' // Or 'memory' (more secure, lost on page refresh)
});
```

### Token Revocation

```javascript
// Revoking a refresh token (e.g., on logout)
await fetch('https://my-tenant.auth0.com/oauth/revoke', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    client_id: 'YOUR_CLIENT_ID',
    token: refreshToken
  })
});
```

> [!note] Access Token Revocation
> JWT access tokens **cannot be revoked** before their expiration (they are stateless). This is why short expiration times are important. If you need immediate revocation, consider using opaque tokens with token introspection, or maintain a server-side deny list.

## 🔗 Sources

- [Access Tokens — Auth0 Docs](https://auth0.com/docs/secure/tokens/access-tokens)
- [Refresh Tokens — Auth0 Docs](https://auth0.com/docs/secure/tokens/refresh-tokens)
- [What Are Refresh Tokens and How to Use Them Securely — Auth0 Blog](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)
- [Understanding Refresh Tokens — Auth0 Learn](https://auth0.com/learn/refresh-tokens)
- [JSON Web Token Claims — Auth0 Docs](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-token-claims)
- [Opaque Versus JWT Access Token — Auth0 Support](https://support.auth0.com/center/s/article/opaque-versus-jwt-access-token)

## 📝 Related Notes

- [[auth0-lesson-01-what-is-auth0]]
- [[auth0-lesson-02-core-concepts]]
- [[auth0-lesson-03-authentication-protocols]]
- [[auth0-lesson-04-auth-flows]]
- [[auth0-lesson-06-jwt-deep-dive]]

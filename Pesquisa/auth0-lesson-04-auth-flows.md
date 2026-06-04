---
title: "Auth0 Lesson 4 — Auth Flows"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - status/rascunho
area: authentication
---

# 🔄 Auth0 Lesson 4 — Auth Flows

## 🎯 Context

Understanding which OAuth 2.0 flow to use for each application type is critical for security. This lesson covers each flow in detail, with step-by-step walkthroughs, sequence diagrams, and code examples.

## 📋 Flow Selection Guide

| Application Type | Recommended Flow | Why |
|-----------------|------------------|-----|
| **Regular Web App** (server-side) | Authorization Code | Server can securely store client secret |
| **SPA** (React, Angular, Vue) | Authorization Code + PKCE | Public client; PKCE prevents code interception |
| **Native/Mobile App** (iOS, Android) | Authorization Code + PKCE | Public client; PKCE prevents code interception |
| **Machine-to-Machine** (backend service) | Client Credentials | No user involved; app authenticates directly |
| **TV / IoT / CLI** (input-constrained) | Device Authorization | No browser on device; user authorizes on another device |
| **SPA** (legacy) | ~~Implicit~~ | **Deprecated** — use Auth Code + PKCE instead |
| **Trusted first-party app** (legacy) | ~~Resource Owner Password~~ | **Deprecated** — use Auth Code or Auth Code + PKCE |

## 1️⃣ Authorization Code Flow

### When to Use

- Server-side applications (Express.js, ASP.NET, Rails, Django, etc.)
- Applications that can **securely store a client secret** on the server

### How It Works — Step by Step

```
┌──────────┐          ┌──────────────┐          ┌──────────────┐
│  User     │          │  Your App     │          │  Auth0        │
│  (Browser)│          │  (Server)     │          │  (AuthZ Srv)  │
└─────┬─────┘          └──────┬───────┘          └──────┬───────┘
      │                       │                         │
      │ 1. Click "Login"      │                         │
      │──────────────────────►│                         │
      │                       │                         │
      │ 2. Redirect to Auth0  │                         │
      │◄──────────────────────│                         │
      │                       │                         │
      │ 3. GET /authorize?response_type=code&...        │
      │────────────────────────────────────────────────►│
      │                       │                         │
      │ 4. Auth0 shows login page                       │
      │◄────────────────────────────────────────────────│
      │                       │                         │
      │ 5. User authenticates │                         │
      │────────────────────────────────────────────────►│
      │                       │                         │
      │ 6. Redirect to callback with ?code=AUTH_CODE    │
      │◄────────────────────────────────────────────────│
      │                       │                         │
      │ 7. Forward code to server                       │
      │──────────────────────►│                         │
      │                       │                         │
      │                       │ 8. POST /oauth/token    │
      │                       │    {code, client_id,    │
      │                       │     client_secret,      │
      │                       │     redirect_uri}       │
      │                       │────────────────────────►│
      │                       │                         │
      │                       │ 9. {access_token,       │
      │                       │     id_token,           │
      │                       │     refresh_token}      │
      │                       │◄────────────────────────│
      │                       │                         │
      │ 10. Set session       │                         │
      │◄──────────────────────│                         │
```

### Step Details

1. User clicks "Login" in your application
2. Your server generates the authorization URL and redirects the browser
3. Browser hits Auth0's `/authorize` endpoint with parameters:
   - `response_type=code`
   - `client_id=YOUR_CLIENT_ID`
   - `redirect_uri=https://yourapp.com/callback`
   - `scope=openid profile email`
   - `state=RANDOM_STATE_VALUE` (CSRF protection)
4. Auth0 displays Universal Login page
5. User enters credentials (or uses social/enterprise login)
6. Auth0 validates credentials and redirects back to `redirect_uri` with an **authorization code** (short-lived, single-use)
7. Browser follows redirect to your server's callback endpoint
8. **Server-side** (not browser): Your server exchanges the code for tokens by calling `/oauth/token` with the `client_secret`
9. Auth0 validates the code + client secret and returns tokens
10. Server creates a session for the user

### Code Example (Node.js / Express)

```javascript
const express = require('express');
const { auth } = require('express-openid-connect');

const app = express();

app.use(
  auth({
    authRequired: false,
    auth0Logout: true,
    issuerBaseURL: 'https://my-tenant.auth0.com',
    baseURL: 'https://myapp.com',
    clientID: 'YOUR_CLIENT_ID',
    clientSecret: 'YOUR_CLIENT_SECRET',
    secret: 'SESSION_ENCRYPTION_SECRET',
    authorizationParams: {
      response_type: 'code',
      scope: 'openid profile email',
      audience: 'https://api.myapp.com'
    }
  })
);
```

## 2️⃣ Authorization Code Flow with PKCE

### When to Use

- **SPAs** (React, Angular, Vue)
- **Native/Mobile apps** (iOS, Android)
- Any **public client** that cannot securely store a client secret

### What is PKCE?

PKCE (Proof Key for Code Exchange, pronounced "pixie") adds a cryptographic challenge to the Authorization Code flow. It prevents **authorization code interception attacks** where a malicious app intercepts the code during the redirect.

### How PKCE Works

```
1. Client generates a random string:        code_verifier
2. Client hashes it:                         code_challenge = SHA256(code_verifier)
3. Client sends code_challenge to /authorize
4. Auth0 stores the code_challenge
5. Auth0 returns authorization code
6. Client sends code + code_verifier to /oauth/token
7. Auth0 computes SHA256(code_verifier) and compares with stored code_challenge
8. If they match → tokens issued
```

> [!important] Why This is Secure
> An attacker who intercepts the authorization code in step 5 **cannot exchange it** because they don't have the `code_verifier` (it was never transmitted in the redirect). They only have the `code_challenge` (a hash), and SHA-256 is irreversible.

### Step-by-Step Flow

```
┌──────────┐          ┌──────────────┐          ┌──────────────┐
│  User     │          │  SPA / Mobile │          │  Auth0        │
│  (Browser)│          │  (Public App) │          │  (AuthZ Srv)  │
└─────┬─────┘          └──────┬───────┘          └──────┬───────┘
      │                       │                         │
      │ 1. Click "Login"      │                         │
      │──────────────────────►│                         │
      │                       │                         │
      │                       │ 2. Generate:            │
      │                       │    code_verifier (rand) │
      │                       │    code_challenge =     │
      │                       │      SHA256(verifier)   │
      │                       │                         │
      │ 3. Redirect to Auth0 with code_challenge        │
      │◄──────────────────────│                         │
      │                       │                         │
      │ 4. GET /authorize?                              │
      │    response_type=code                           │
      │    &code_challenge=...                          │
      │    &code_challenge_method=S256                  │
      │────────────────────────────────────────────────►│
      │                       │                         │
      │ 5. Login page         │                         │
      │◄───────────────────────────────────────────────►│
      │                       │                         │
      │ 6. Redirect with ?code=AUTH_CODE                │
      │◄────────────────────────────────────────────────│
      │                       │                         │
      │ 7. Forward code       │                         │
      │──────────────────────►│                         │
      │                       │                         │
      │                       │ 8. POST /oauth/token    │
      │                       │    {code,               │
      │                       │     code_verifier,      │
      │                       │     client_id}          │
      │                       │    (NO client_secret!)  │
      │                       │────────────────────────►│
      │                       │                         │
      │                       │ 9. Auth0 verifies:      │
      │                       │    SHA256(code_verifier) │
      │                       │    == stored challenge  │
      │                       │                         │
      │                       │ 10. {access_token,      │
      │                       │      id_token}          │
      │                       │◄────────────────────────│
```

### Code Example (React SPA)

```javascript
import { Auth0Provider, useAuth0 } from '@auth0/auth0-react';

// In your app root:
<Auth0Provider
  domain="my-tenant.auth0.com"
  clientId="YOUR_CLIENT_ID"
  authorizationParams={{
    redirect_uri: window.location.origin,
    audience: 'https://api.myapp.com',
    scope: 'openid profile email read:data'
  }}
>
  <App />
</Auth0Provider>

// In a component:
function LoginButton() {
  const { loginWithRedirect, getAccessTokenSilently } = useAuth0();

  const callApi = async () => {
    const token = await getAccessTokenSilently();
    // The SDK handles PKCE automatically
    const response = await fetch('https://api.myapp.com/data', {
      headers: { Authorization: `Bearer ${token}` }
    });
  };

  return <button onClick={() => loginWithRedirect()}>Log In</button>;
}
```

> [!note] SDK Handles PKCE
> Auth0's SDKs (auth0-spa-js, auth0-react, etc.) handle the entire PKCE flow automatically — generating `code_verifier`, computing `code_challenge`, and exchanging the code. You don't need to implement PKCE manually.

### PKCE Technical Details

- **code_verifier**: Random string, 43-128 characters, using `[A-Z] / [a-z] / [0-9] / "-" / "." / "_" / "~"`
- **code_challenge**: `BASE64URL(SHA256(code_verifier))`
- **code_challenge_method**: Always `S256` (Auth0 only supports S256, not `plain`)

```javascript
// Manual PKCE generation (for understanding — SDKs do this for you)
import crypto from 'crypto';

function generateCodeVerifier() {
  return crypto.randomBytes(32)
    .toString('base64url');  // 43 characters
}

function generateCodeChallenge(verifier) {
  return crypto.createHash('sha256')
    .update(verifier)
    .digest('base64url');
}

const codeVerifier = generateCodeVerifier();
const codeChallenge = generateCodeChallenge(codeVerifier);
```

## 3️⃣ Client Credentials Flow

### When to Use

- **Machine-to-Machine (M2M)** communication
- Backend services calling other backend APIs
- Cron jobs, daemons, CLI tools
- Microservice-to-microservice authentication
- **No user is involved** — the application authenticates as itself

### How It Works

This is the simplest flow — a single POST request:

```
┌──────────────┐          ┌──────────────┐
│  Your Service │          │  Auth0        │
│  (M2M Client) │          │  (AuthZ Srv)  │
└──────┬───────┘          └──────┬───────┘
       │                         │
       │ 1. POST /oauth/token    │
       │    {client_id,          │
       │     client_secret,      │
       │     audience,           │
       │     grant_type:         │
       │     "client_credentials"│
       │    }                    │
       │────────────────────────►│
       │                         │
       │ 2. Auth0 validates      │
       │    client credentials   │
       │    and scopes           │
       │                         │
       │ 3. {access_token}       │
       │◄────────────────────────│
       │                         │
       │ 4. Call API with        │
       │    Bearer token         │
       │────────────────────────►│ Resource Server
       │                         │
```

### Code Example (Node.js)

```javascript
const axios = require('axios');

async function getM2MToken() {
  const response = await axios.post(
    'https://my-tenant.auth0.com/oauth/token',
    {
      client_id: 'YOUR_M2M_CLIENT_ID',
      client_secret: 'YOUR_M2M_CLIENT_SECRET',
      audience: 'https://api.myapp.com',
      grant_type: 'client_credentials'
    },
    {
      headers: { 'Content-Type': 'application/json' }
    }
  );

  return response.data.access_token;
}

// Use the token
async function callProtectedApi() {
  const token = await getM2MToken();

  const response = await axios.get('https://api.myapp.com/data', {
    headers: { Authorization: `Bearer ${token}` }
  });

  return response.data;
}
```

### Key Points

- Returns **only an access token** (no ID token, no refresh token — there's no user)
- The access token contains **scopes** that define what the M2M app can do
- Scopes must be configured in the Auth0 Dashboard under the API's **Machine-to-Machine Applications** tab
- **Cache the token** until it expires; don't request a new one for every API call

## 4️⃣ Device Authorization Flow

### When to Use

- **Input-constrained devices**: Smart TVs, game consoles, IoT devices, printers
- **CLI tools** where opening a browser is awkward
- Any device where typing credentials is difficult or impossible

### How It Works

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Device       │     │  Auth0        │     │  User         │
│  (TV/IoT/CLI) │     │  (AuthZ Srv)  │     │  (Phone/PC)   │
└──────┬───────┘     └──────┬───────┘     └──────┬───────┘
       │                    │                     │
       │ 1. POST            │                     │
       │ /oauth/device/code │                     │
       │ {client_id, scope} │                     │
       │───────────────────►│                     │
       │                    │                     │
       │ 2. Returns:        │                     │
       │ {device_code,      │                     │
       │  user_code: "HGXF",│                     │
       │  verification_uri, │                     │
       │  interval: 5}      │                     │
       │◄───────────────────│                     │
       │                    │                     │
       │ 3. Display to user:│                     │
       │ "Go to             │                     │
       │  https://auth0.com/activate              │
       │  Enter code: HGXF" │                     │
       │                    │                     │
       │                    │  4. User visits URL │
       │                    │  and enters code    │
       │                    │◄────────────────────│
       │                    │                     │
       │                    │  5. User logs in    │
       │                    │  and consents       │
       │                    │◄───────────────────►│
       │                    │                     │
       │ 6. Device polls:   │                     │
       │ POST /oauth/token  │                     │
       │ {device_code,      │                     │
       │  grant_type:       │                     │
       │  "device_code"}    │                     │
       │───────────────────►│                     │
       │                    │                     │
       │ (If not yet        │                     │
       │  authorized:       │                     │
       │  "authorization_   │                     │
       │   pending")        │                     │
       │◄───────────────────│                     │
       │                    │                     │
       │ 7. Once authorized:│                     │
       │ {access_token,     │                     │
       │  id_token}         │                     │
       │◄───────────────────│                     │
```

### Polling Behavior

The device must poll the token endpoint at the `interval` (in seconds) returned in step 2. Possible responses during polling:

| Response | Meaning |
|----------|---------|
| `authorization_pending` | User hasn't authorized yet; keep polling |
| `slow_down` | Polling too fast; increase interval by 5 seconds |
| `expired_token` | The `device_code` expired; start over |
| `access_denied` | User denied authorization |
| `{ access_token, ... }` | Success! |

### Code Example (Node.js CLI)

```javascript
const axios = require('axios');

async function deviceLogin() {
  // Step 1: Request device code
  const { data } = await axios.post(
    'https://my-tenant.auth0.com/oauth/device/code',
    {
      client_id: 'YOUR_CLIENT_ID',
      scope: 'openid profile',
      audience: 'https://api.myapp.com'
    }
  );

  console.log(`Go to: ${data.verification_uri_complete}`);
  console.log(`Or visit ${data.verification_uri} and enter code: ${data.user_code}`);

  // Step 2: Poll for token
  const interval = data.interval * 1000; // convert to ms
  while (true) {
    await new Promise(resolve => setTimeout(resolve, interval));

    try {
      const tokenResponse = await axios.post(
        'https://my-tenant.auth0.com/oauth/token',
        {
          grant_type: 'urn:ietf:params:oauth:grant-type:device_code',
          device_code: data.device_code,
          client_id: 'YOUR_CLIENT_ID'
        }
      );
      return tokenResponse.data; // { access_token, id_token, ... }
    } catch (err) {
      if (err.response?.data?.error === 'authorization_pending') continue;
      if (err.response?.data?.error === 'slow_down') {
        interval += 5000;
        continue;
      }
      throw err; // expired_token, access_denied, or other error
    }
  }
}
```

## 5️⃣ Implicit Flow (Legacy — Deprecated)

### What It Was

The Implicit Flow was designed for SPAs before PKCE existed. It returned tokens **directly in the URL fragment** (after the `#`):

```
https://yourapp.com/callback#access_token=eyJ...&id_token=eyJ...&token_type=Bearer
```

### Why It's Deprecated

| Problem | Details |
|---------|---------|
| **Token exposure in URL** | Access tokens visible in browser history, referrer headers, and server logs |
| **No refresh tokens** | Implicit flow cannot return refresh tokens |
| **Token leakage** | Browser extensions, analytics tools, and referrer headers can capture the token |
| **No code exchange** | No opportunity for server-side validation |

### What Replaced It

**Authorization Code Flow with PKCE** completely replaces the Implicit Flow for SPAs and native apps. PKCE provides:
- Tokens are exchanged via a secure backend channel (POST to `/oauth/token`)
- Tokens never appear in URLs
- Supports refresh tokens
- Code interception is prevented by the PKCE challenge

> [!warning] Migration Required
> If you have applications still using the Implicit Flow, migrate them to Authorization Code + PKCE. Auth0 still supports Implicit Flow for backward compatibility, but it should not be used for new applications.

### Implicit Flow with Form Post (Auth0 Variant)

Auth0 offers a slightly more secure variant where the token is returned via an HTTP POST to the callback URL (instead of a URL fragment). This is sometimes used when you only need an **ID Token** for authentication and don't need an access token:

```
response_type=id_token&response_mode=form_post
```

This is acceptable for simple authentication scenarios where you only need to know who the user is, but for API access, always use Authorization Code + PKCE.

## 6️⃣ Resource Owner Password Grant (Legacy — Deprecated)

### What It Was

The application collects the user's username and password **directly** and sends them to Auth0:

```bash
# DO NOT USE IN PRODUCTION
curl --request POST \
  --url 'https://my-tenant.auth0.com/oauth/token' \
  --header 'content-type: application/json' \
  --data '{
    "grant_type": "password",
    "username": "alice@example.com",
    "password": "hunter2",
    "client_id": "YOUR_CLIENT_ID",
    "client_secret": "YOUR_CLIENT_SECRET",
    "audience": "https://api.myapp.com",
    "scope": "openid profile email"
  }'
```

### Why It's Discouraged (and Deprecated)

| Problem | Details |
|---------|---------|
| **User credentials exposed to the app** | The whole point of OAuth is that the app should **never see** the user's password |
| **No MFA support** | Cannot trigger multi-factor authentication |
| **No social/enterprise login** | Only works with database connections |
| **No consent screen** | User cannot review what they're authorizing |
| **Deprecated in OAuth 2.1** | Officially removed from the spec |
| **Auth0 migration** | Auth0 deprecated the legacy `/oauth/ro` endpoint in 2017; the `/oauth/token` endpoint still accepts it but it's discouraged |

### What Replaced It

- For **input-constrained devices**: Use the **Device Authorization Flow**
- For **first-party apps**: Use **Authorization Code + PKCE**
- For **testing/development**: Use the Authorization Code flow with Auth0's test tools

## 📊 Summary: Which Flow for Which App

```
                    ┌─────────────────────────┐
                    │   Is a user involved?    │
                    └────────────┬────────────┘
                         ┌───────┴───────┐
                        Yes              No
                         │                │
                ┌────────▼────────┐      │
                │ Can the app open │      │
                │ a browser?       │      │
                └───────┬─────────┘      │
               ┌────────┴────────┐       │
              Yes                No      │
               │                  │       │
       ┌───────▼────────┐   ┌────▼───┐  │
       │ Can it store a  │   │ Device │  │
       │ client secret?  │   │ Flow   │  │
       └───────┬─────────┘   └────────┘  │
          ┌────┴────┐                     │
         Yes        No                   │
          │          │                    │
     ┌────▼─────┐  ┌▼──────────┐  ┌─────▼──────────┐
     │ Auth Code │  │ Auth Code  │  │ Client          │
     │ Flow      │  │ + PKCE     │  │ Credentials     │
     └──────────┘  └───────────┘  └────────────────┘
```

## 🔗 Sources

- [Authorization Code Flow with PKCE — Auth0 Docs](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce)
- [Client Credentials Flow — Auth0 Docs](https://auth0.com/docs/get-started/authentication-and-authorization-flow/client-credentials-flow)
- [Device Authorization Flow — Auth0 Docs](https://dev.auth0.com/docs/get-started/authentication-and-authorization-flow/device-authorization-flow)
- [Which OAuth Flow to Use — Auth0 GitHub Docs](https://github.com/auth0/docs/blob/master/articles/api-auth/which-oauth-flow-to-use.md)
- [Implicit Flow is Dead — Logto Blog](https://blog.logto.io/implicit-flow-is-dead)
- [OAuth Grant Types Explained — SuperTokens](https://supertokens.com/blog/oauth-grant-types-explained)
- [Using Machine to Machine (M2M) Authorization — Auth0 Blog](https://auth0.com/blog/using-m2m-authorization/)

## 📝 Related Notes

- [[auth0-lesson-01-what-is-auth0]]
- [[auth0-lesson-02-core-concepts]]
- [[auth0-lesson-03-authentication-protocols]]
- [[auth0-lesson-05-tokens]]
- [[auth0-lesson-06-jwt-deep-dive]]

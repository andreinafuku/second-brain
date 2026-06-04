---
title: "Auth0 Lesson 8 — Hands-on: Web App Integration"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - auth0
  - identity
  - status/rascunho
area: arquitetura
---

# 🌐 Auth0 Lesson 8 — Hands-on: Web App Integration

## 🎯 Overview

This lesson covers the practical integration of Auth0 into web applications. We'll walk through Node.js/Express (server-side), React SPA (client-side), and Next.js (full-stack) integrations, using the official Auth0 SDKs. Each section includes step-by-step code and explanations.

---

## 🟢 Integrating Auth0 in a Node.js Express App

The official SDK is **`express-openid-connect`** — an Express.js middleware that handles OpenID Connect authentication.

### Prerequisites

- Node.js 18+ and npm 10+
- Express 4.17.0+
- An Auth0 **Regular Web Application** registered (see [[auth0-lesson-07-hands-on-setup]])

### Step 1: Create the Project

```bash
mkdir auth0-express-app && cd auth0-express-app
npm init -y
npm install express express-openid-connect dotenv
npm install --save-dev nodemon
```

### Step 2: Configure Environment Variables

Create a `.env` file:

```bash
# Generate a secure secret
openssl rand -hex 32

# .env file
ISSUER_BASE_URL=https://YOUR_AUTH0_DOMAIN
CLIENT_ID=YOUR_CLIENT_ID
SECRET=YOUR_GENERATED_SECRET_AT_LEAST_32_CHARS
BASE_URL=http://localhost:3000
```

### Step 3: Basic Server with Authentication

```javascript
// index.js
require('dotenv').config();
const express = require('express');
const { auth } = require('express-openid-connect');

const app = express();
const port = process.env.PORT || 3000;

// Auth0 configuration
const config = {
  authRequired: false,        // false = allow unauthenticated access to some routes
  auth0Logout: true,          // true = logout from Auth0 as well (not just the app)
  secret: process.env.SECRET,
  baseURL: process.env.BASE_URL,
  clientID: process.env.CLIENT_ID,
  issuerBaseURL: process.env.ISSUER_BASE_URL,
};

// The auth() middleware automatically creates these routes:
//   /login    — redirects to Auth0 Universal Login
//   /logout   — clears session and redirects to Auth0 logout
//   /callback — handles the Auth0 redirect after login
app.use(auth(config));

// Public home route
app.get('/', (req, res) => {
  const isAuthenticated = req.oidc.isAuthenticated();
  res.send(`
    <h1>Auth0 Express App</h1>
    <p>Status: ${isAuthenticated ? 'Logged In' : 'Logged Out'}</p>
    ${isAuthenticated
      ? '<a href="/profile">Profile</a> | <a href="/logout">Logout</a>'
      : '<a href="/login">Login</a>'}
  `);
});

app.listen(port, () => {
  console.log(`Server running at http://localhost:${port}`);
});
```

### Step 4: Add Protected Routes

```javascript
const { auth, requiresAuth } = require('express-openid-connect');

// ... (auth middleware setup from above)

// Protected route — requires authentication
app.get('/profile', requiresAuth(), (req, res) => {
  const user = req.oidc.user;
  res.send(`
    <h1>User Profile</h1>
    <img src="${user.picture}" width="80" />
    <h2>${user.name}</h2>
    <p>Email: ${user.email}</p>
    <p>Sub (user ID): ${user.sub}</p>
    <pre>${JSON.stringify(user, null, 2)}</pre>
    <a href="/">Home</a> | <a href="/logout">Logout</a>
  `);
});

// Protect an entire router
const protectedRouter = express.Router();
protectedRouter.use(requiresAuth());

protectedRouter.get('/dashboard', (req, res) => {
  res.send(`Welcome to the dashboard, ${req.oidc.user.name}!`);
});

protectedRouter.get('/settings', (req, res) => {
  res.send('Settings page');
});

app.use('/app', protectedRouter);
// Now /app/dashboard and /app/settings both require auth
```

### Step 5: Getting User Info After Login

```javascript
// The req.oidc.user object contains claims from the ID token:
app.get('/profile', requiresAuth(), (req, res) => {
  const user = req.oidc.user;

  // Standard OIDC claims:
  console.log(user.sub);         // Unique user ID (e.g., "auth0|abc123")
  console.log(user.name);        // Full name
  console.log(user.email);       // Email address
  console.log(user.picture);     // Profile picture URL
  console.log(user.email_verified); // Boolean
  console.log(user.nickname);    // Nickname
  console.log(user.updated_at);  // Last profile update timestamp

  res.json(user);
});

// To get additional user info (app_metadata, user_metadata, etc.),
// use the /userinfo endpoint or the Management API
app.get('/detailed-profile', requiresAuth(), async (req, res) => {
  const userInfo = await req.oidc.fetchUserInfo();
  res.json(userInfo);
});
```

### Step 6: Calling a Protected API with Access Tokens

To call your own API, configure the SDK to request an access token:

```javascript
const config = {
  authRequired: false,
  auth0Logout: true,
  secret: process.env.SECRET,
  baseURL: process.env.BASE_URL,
  clientID: process.env.CLIENT_ID,
  issuerBaseURL: process.env.ISSUER_BASE_URL,
  // Add these to request an access token for your API:
  clientSecret: process.env.CLIENT_SECRET,
  authorizationParams: {
    response_type: 'code',
    audience: 'https://my-api.example.com',  // Your API identifier in Auth0
    scope: 'openid profile email read:data',
  },
};

// Get access token and call API
app.get('/call-api', requiresAuth(), async (req, res) => {
  try {
    const { access_token } = req.oidc.accessToken;

    const response = await fetch('https://my-api.example.com/data', {
      headers: {
        Authorization: `Bearer ${access_token}`,
      },
    });

    const data = await response.json();
    res.json(data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

### Claim-Based Authorization

```javascript
const { claimEquals, claimIncludes } = require('express-openid-connect');

// Only users with role "admin"
app.get('/admin', claimEquals('role', 'admin'), (req, res) => {
  res.send('Admin panel');
});

// Users with any of the specified permissions
app.get('/reports', claimIncludes('permissions', 'read:reports'), (req, res) => {
  res.send('Reports page');
});
```

---

## ⚛️ Integrating Auth0 in a React SPA

The official SDK is **`@auth0/auth0-react`** — a React wrapper around `@auth0/auth0-spa-js`.

### Prerequisites

- React 18+ (Vite or CRA)
- An Auth0 **Single Page Application** registered

### Step 1: Create the Project

```bash
npm create vite@latest auth0-react-app -- --template react-ts
cd auth0-react-app
npm install @auth0/auth0-react
npm install react-router-dom
```

### Step 2: Configure Environment Variables

Create `.env`:

```bash
VITE_AUTH0_DOMAIN=your-tenant.us.auth0.com
VITE_AUTH0_CLIENT_ID=your_client_id
VITE_AUTH0_AUDIENCE=https://my-api.example.com  # optional, for API calls
```

### Step 3: Auth0Provider Setup

```tsx
// src/main.tsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Auth0Provider } from '@auth0/auth0-react';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <BrowserRouter>
      <Auth0Provider
        domain={import.meta.env.VITE_AUTH0_DOMAIN}
        clientId={import.meta.env.VITE_AUTH0_CLIENT_ID}
        authorizationParams={{
          redirect_uri: window.location.origin,
          audience: import.meta.env.VITE_AUTH0_AUDIENCE, // optional
          scope: 'openid profile email',
        }}
      >
        <App />
      </Auth0Provider>
    </BrowserRouter>
  </React.StrictMode>
);
```

### Step 4: Login and Logout Buttons

```tsx
// src/components/LoginButton.tsx
import { useAuth0 } from '@auth0/auth0-react';

export const LoginButton = () => {
  const { loginWithRedirect } = useAuth0();
  return <button onClick={() => loginWithRedirect()}>Log In</button>;
};

// src/components/LogoutButton.tsx
import { useAuth0 } from '@auth0/auth0-react';

export const LogoutButton = () => {
  const { logout } = useAuth0();
  return (
    <button onClick={() => logout({
      logoutParams: { returnTo: window.location.origin }
    })}>
      Log Out
    </button>
  );
};

// src/components/AuthButton.tsx — Conditional button
import { useAuth0 } from '@auth0/auth0-react';
import { LoginButton } from './LoginButton';
import { LogoutButton } from './LogoutButton';

export const AuthButton = () => {
  const { isAuthenticated, isLoading } = useAuth0();
  if (isLoading) return <span>Loading...</span>;
  return isAuthenticated ? <LogoutButton /> : <LoginButton />;
};
```

### Step 5: Display User Profile

```tsx
// src/components/Profile.tsx
import { useAuth0 } from '@auth0/auth0-react';

export const Profile = () => {
  const { user, isAuthenticated, isLoading } = useAuth0();

  if (isLoading) return <div>Loading...</div>;
  if (!isAuthenticated || !user) return null;

  return (
    <div>
      <img src={user.picture} alt={user.name} width={80} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <p>User ID: {user.sub}</p>
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </div>
  );
};
```

### Step 6: Protected Routes

```tsx
// src/components/ProtectedRoute.tsx
import { useAuth0, withAuthenticationRequired } from '@auth0/auth0-react';
import { Navigate } from 'react-router-dom';

// Option A: Using the hook
export const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const { isAuthenticated, isLoading, loginWithRedirect } = useAuth0();

  if (isLoading) return <div>Loading...</div>;

  if (!isAuthenticated) {
    loginWithRedirect();
    return <div>Redirecting to login...</div>;
  }

  return <>{children}</>;
};

// Option B: Using the HOC (recommended by Auth0)
export const ProtectedPage = withAuthenticationRequired(
  () => <div>This is a protected page!</div>,
  {
    onRedirecting: () => <div>Redirecting you to the login page...</div>,
  }
);
```

Using protected routes in your app:

```tsx
// src/App.tsx
import { Routes, Route } from 'react-router-dom';
import { ProtectedRoute, ProtectedPage } from './components/ProtectedRoute';
import { Profile } from './components/Profile';
import { AuthButton } from './components/AuthButton';

function App() {
  return (
    <div>
      <nav>
        <AuthButton />
      </nav>
      <Routes>
        <Route path="/" element={<h1>Home (Public)</h1>} />
        <Route
          path="/profile"
          element={
            <ProtectedRoute>
              <Profile />
            </ProtectedRoute>
          }
        />
        <Route path="/dashboard" element={<ProtectedPage />} />
      </Routes>
    </div>
  );
}
```

### Step 7: Calling a Protected API with Access Tokens

```tsx
// src/hooks/useApi.ts
import { useAuth0 } from '@auth0/auth0-react';
import { useState, useCallback } from 'react';

export const useApi = (url: string) => {
  const { getAccessTokenSilently } = useAuth0();
  const [data, setData] = useState<any>(null);
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);

  const callApi = useCallback(async () => {
    setLoading(true);
    try {
      const token = await getAccessTokenSilently();
      const response = await fetch(url, {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });
      const responseData = await response.json();
      setData(responseData);
    } catch (err: any) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, [getAccessTokenSilently, url]);

  return { data, error, loading, callApi };
};

// Usage in a component:
const Dashboard = () => {
  const { data, error, loading, callApi } = useApi('https://my-api.example.com/data');

  return (
    <div>
      <button onClick={callApi} disabled={loading}>
        {loading ? 'Loading...' : 'Fetch Data'}
      </button>
      {error && <p>Error: {error}</p>}
      {data && <pre>{JSON.stringify(data, null, 2)}</pre>}
    </div>
  );
};
```

---

## 🔷 Using nextjs-auth0 (Next.js SDK)

The official SDK for Next.js is **`@auth0/nextjs-auth0`** — it supports both App Router and Pages Router.

### Step 1: Install and Configure

```bash
npm install @auth0/nextjs-auth0
```

Create `.env.local`:

```bash
AUTH0_SECRET=use-a-long-random-string-at-least-32-chars
AUTH0_BASE_URL=http://localhost:3000
AUTH0_ISSUER_BASE_URL=https://YOUR_AUTH0_DOMAIN
AUTH0_CLIENT_ID=YOUR_CLIENT_ID
AUTH0_CLIENT_SECRET=YOUR_CLIENT_SECRET
```

### Step 2: Create Auth Route Handler (App Router)

```typescript
// app/api/auth/[auth0]/route.ts
import { handleAuth } from '@auth0/nextjs-auth0';

export const GET = handleAuth();
// This creates: /api/auth/login, /api/auth/logout, /api/auth/callback, /api/auth/me
```

### Step 3: Wrap Your App with Auth0Provider

```tsx
// app/layout.tsx
import { Auth0Provider } from '@auth0/nextjs-auth0';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <Auth0Provider>
          {children}
        </Auth0Provider>
      </body>
    </html>
  );
}
```

### Step 4: Server-Side Authentication (Server Components)

```tsx
// app/profile/page.tsx (Server Component)
import { getSession } from '@auth0/nextjs-auth0';
import { redirect } from 'next/navigation';

export default async function ProfilePage() {
  const session = await getSession();

  if (!session) {
    redirect('/api/auth/login');
  }

  const { user } = session;

  return (
    <div>
      <h1>Profile</h1>
      <img src={user.picture} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### Step 5: Client-Side Authentication (Client Components)

```tsx
'use client';
// app/components/ClientProfile.tsx
import { useUser } from '@auth0/nextjs-auth0';

export default function ClientProfile() {
  const { user, error, isLoading } = useUser();

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>{error.message}</div>;
  if (!user) return <a href="/api/auth/login">Login</a>;

  return (
    <div>
      <p>Welcome, {user.name}!</p>
      <a href="/api/auth/logout">Logout</a>
    </div>
  );
}
```

### Step 6: Middleware Protection

```typescript
// middleware.ts
import { withMiddlewareAuthRequired } from '@auth0/nextjs-auth0/edge';

export default withMiddlewareAuthRequired();

export const config = {
  matcher: ['/dashboard/:path*', '/profile/:path*', '/settings/:path*'],
};
```

### Step 7: Calling APIs from Next.js

```typescript
// app/api/external/route.ts (API Route — server-side)
import { getAccessToken } from '@auth0/nextjs-auth0';
import { NextResponse } from 'next/server';

export async function GET() {
  try {
    const { accessToken } = await getAccessToken();

    const response = await fetch('https://my-api.example.com/data', {
      headers: {
        Authorization: `Bearer ${accessToken}`,
      },
    });

    const data = await response.json();
    return NextResponse.json(data);
  } catch (error: any) {
    return NextResponse.json({ error: error.message }, { status: 500 });
  }
}
```

---

## 🛡️ Protecting Your Own API (Backend)

If you are building an API that receives access tokens from Auth0, you need to validate them.

### Step 1: Register Your API in Auth0

1. Go to **Dashboard > Applications > APIs**
2. Click **"+ Create API"**
3. Enter:
   - **Name**: "My API"
   - **Identifier** (Audience): `https://my-api.example.com` (a logical URI, not an actual URL)
   - **Signing Algorithm**: RS256
4. Configure **Permissions** (e.g., `read:data`, `write:data`)

### Step 2: Validate Tokens in Express

```bash
npm install express-oauth2-jwt-bearer
```

```javascript
// api/server.js
const express = require('express');
const { auth, requiredScopes } = require('express-oauth2-jwt-bearer');

const app = express();

// JWT validation middleware
const checkJwt = auth({
  audience: 'https://my-api.example.com',
  issuerBaseURL: 'https://YOUR_AUTH0_DOMAIN',
  tokenSigningAlg: 'RS256',
});

// Public endpoint
app.get('/api/public', (req, res) => {
  res.json({ message: 'This is public' });
});

// Protected endpoint — requires valid access token
app.get('/api/private', checkJwt, (req, res) => {
  res.json({
    message: 'This is private',
    user: req.auth.payload,  // Decoded token payload
  });
});

// Protected endpoint — requires specific scopes
app.get('/api/private-scoped', checkJwt, requiredScopes('read:data'), (req, res) => {
  res.json({ message: 'This requires read:data scope' });
});

// Extract user info from the token
app.get('/api/me', checkJwt, (req, res) => {
  const { sub, permissions, scope } = req.auth.payload;
  res.json({
    userId: sub,
    permissions,
    scope,
  });
});

app.listen(3001, () => console.log('API running on port 3001'));
```

---

## 📊 SDK Comparison

| Feature | `express-openid-connect` | `@auth0/auth0-react` | `@auth0/nextjs-auth0` |
|---------|--------------------------|----------------------|------------------------|
| **App Type** | Regular Web App (Express) | SPA (React) | Full-Stack (Next.js) |
| **Token Storage** | Server-side session (cookie) | In-memory (browser) | Server-side session (cookie) |
| **Security Level** | High (confidential client) | Medium (public client) | High (confidential client) |
| **OAuth Flow** | Authorization Code | Authorization Code + PKCE | Authorization Code |
| **Auto Routes** | `/login`, `/logout`, `/callback` | N/A (hooks) | `/api/auth/*` |
| **SSR Support** | Yes | No | Yes |
| **Middleware** | `requiresAuth()` | `withAuthenticationRequired` | `withMiddlewareAuthRequired` |

---

## ⚠️ Common Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| "Callback URL mismatch" | redirect_uri not in allowed list | Add exact URL to Allowed Callback URLs |
| `req.oidc` is undefined | `auth()` middleware not registered | Ensure `app.use(auth(config))` before routes |
| Silent token renewal fails | Missing Allowed Web Origins | Add your origin to Allowed Web Origins |
| CORS errors | Origin not configured | Add origin to Allowed Origins (CORS) |
| "Login required" on API call | Token expired or not present | Check `getAccessTokenSilently()` errors |
| Infinite redirect loop | `authRequired: true` with no `/login` handler | Set `authRequired: false` or check config |

---

## 🔗 Related Notes

- [[auth0-lesson-07-hands-on-setup]]
- [[auth0-lesson-09-rules-actions-hooks]]
- [[auth0-lesson-10-roles-permissions-rbac]]
- [[auth0-lesson-11-security-best-practices]]
- [[auth0-lesson-12-social-enterprise-connections]]

## 📖 Sources

- [Auth0 Express Quickstart](https://auth0.com/docs/quickstart/webapp/express/interactive)
- [Auth0 React SPA Quickstart](https://auth0.com/docs/quickstart/spa/react/interactive)
- [Auth0 Next.js Quickstart](https://auth0.com/docs/quickstart/webapp/nextjs/interactive)
- [express-openid-connect GitHub](https://github.com/auth0/express-openid-connect)
- [auth0-react GitHub](https://github.com/auth0/auth0-react)
- [nextjs-auth0 GitHub](https://github.com/auth0/nextjs-auth0)
- [express-oauth2-jwt-bearer](https://github.com/auth0/node-oauth2-jwt-bearer)
- [Auth0 React SPA Call an API](https://dev.auth0.com/docs/quickstart/spa/react/02-calling-an-api)

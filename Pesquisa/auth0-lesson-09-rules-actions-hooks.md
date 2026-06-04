---
title: "Auth0 Lesson 9 — Rules, Actions & Hooks"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - auth0
  - identity
  - status/rascunho
area: arquitetura
---

# ⚡ Auth0 Lesson 9 — Rules, Actions & Hooks

## 🎯 Overview

Auth0 provides extensibility points that allow you to run custom code during the authentication pipeline. Over time, these mechanisms evolved:

- **Rules** (legacy, deprecated Nov 2024) — ran after login, before tokens were issued
- **Hooks** (deprecated, end of life Nov 2024) — ran at specific extensibility points beyond login
- **Actions** (current, GA since 2021) — the modern replacement for both Rules and Hooks

Since **November 18, 2024**, Rules and Hooks have reached their **end of life**. New tenants created after October 16, 2023 only support Actions. All existing tenants should migrate to Actions.

---

## 📜 What Are Rules? (Legacy)

Rules were serverless JavaScript functions that executed in a pipeline **after successful authentication** but **before tokens were issued**. They received three parameters:

```javascript
// Rule signature (DEPRECATED)
function myRule(user, context, callback) {
  // user — the authenticated user object
  // context — information about the auth request (client, connection, protocol)
  // callback — call to continue the pipeline or return an error

  // Example: Add a custom claim
  const namespace = 'https://myapp.example.com/';
  context.idToken[namespace + 'role'] = 'admin';
  context.accessToken[namespace + 'role'] = 'admin';

  callback(null, user, context);
}
```

### Rules Characteristics

- **Only post-login**: Rules only ran after authentication
- **Sequential execution**: Rules ran in order (top to bottom in the Dashboard)
- **Shared state**: Rules could pass data via `context` to subsequent Rules
- **Global configuration**: All Rules shared the same configuration object
- **No versioning**: Changes were immediately live
- **Rate limits**: Management API calls from Rules were rate-limited
- **Synchronous pipeline**: Each Rule had to call `callback()` to continue

### Common Rule Patterns

```javascript
// Deny access based on email domain
function denyAccess(user, context, callback) {
  if (user.email && !user.email.endsWith('@mycompany.com')) {
    return callback(new UnauthorizedError('Access denied.'));
  }
  callback(null, user, context);
}

// Enrich tokens with roles from app_metadata
function enrichToken(user, context, callback) {
  const namespace = 'https://myapp.com/';
  const roles = (user.app_metadata && user.app_metadata.roles) || [];
  context.idToken[namespace + 'roles'] = roles;
  context.accessToken[namespace + 'roles'] = roles;
  callback(null, user, context);
}

// Force MFA for specific app
function forceMFA(user, context, callback) {
  if (context.clientID === 'SENSITIVE_APP_CLIENT_ID') {
    context.multifactor = {
      provider: 'any',
      allowRememberBrowser: false,
    };
  }
  callback(null, user, context);
}
```

---

## ⚡ What Are Actions? (The Modern Replacement)

Actions are **secure, tenant-specific, versioned, self-contained functions** that execute at specific points in the Auth0 pipeline. They are written in Node.js (Node 18 runtime) and have isolated execution contexts.

### Key Advantages Over Rules

| Feature | Rules | Actions |
|---------|-------|---------|
| **Triggers** | Post-login only | Multiple (login, M2M, registration, etc.) |
| **Runtime** | Node 12 | Node 18 |
| **Versioning** | No | Yes (draft, deployed, previous versions) |
| **Dependencies** | Limited, shared | Per-action npm packages |
| **Secrets** | Global configuration | Per-action secrets |
| **Testing** | No built-in | Built-in test runner |
| **State sharing** | Via context object | Isolated (no shared state) |
| **Editor** | Basic | Full IDE with autocomplete |
| **Marketplace** | No | Yes (pre-built Actions) |
| **Deployment** | Instant | Explicit deploy step |

### Available Triggers

| Trigger | Event | Execution | Function Name |
|---------|-------|-----------|---------------|
| **Login / Post-Login** | After successful authentication | Blocking (sync) | `onExecutePostLogin` |
| **Machine-to-Machine** | Client Credentials grant | Blocking (sync) | `onExecuteCredentialsExchange` |
| **Pre User Registration** | Before a new user is created | Blocking (sync) | `onExecutePreUserRegistration` |
| **Post User Registration** | After a new user is created | Non-blocking (async) | `onExecutePostUserRegistration` |
| **Post Change Password** | After password is changed | Non-blocking (async) | `onExecutePostChangePassword` |
| **Send Phone Message** | When SMS/Voice MFA is triggered | Blocking (sync) | `onExecuteSendPhoneMessage` |
| **Password Reset Post Challenge** | After password reset verification | Blocking (sync) | `onExecutePostChallenge` |
| **Custom Email Provider** | When Auth0 needs to send an email | Blocking (sync) | `onExecuteCustomEmailProvider` |
| **Custom Phone Provider** | When Auth0 needs to send SMS | Blocking (sync) | `onExecuteCustomPhoneProvider` |

> [!info] Blocking vs Non-Blocking
> **Blocking** triggers pause the Auth0 pipeline until the Action completes. If the Action fails or times out, the pipeline fails.
> **Non-blocking** triggers run asynchronously — the pipeline continues regardless of the Action's outcome.

---

## ✍️ How to Write an Action

### Step 1: Create an Action

1. Go to **Dashboard > Actions > Library**
2. Click **"+ Build Custom"**
3. Enter a name (e.g., "Enrich Token with Roles")
4. Select a **Trigger** (e.g., "Login / Post Login")
5. Select **Runtime** (Node 18)
6. Click **Create**

### Step 2: Write the Code

The Action editor opens with a boilerplate:

```javascript
/**
 * Handler that will be called during the execution of a PostLogin flow.
 *
 * @param {Event} event - Details about the user and the context.
 * @param {PostLoginAPI} api - Interface to change the login behavior.
 */
exports.onExecutePostLogin = async (event, api) => {
  // Your custom logic here
};

/**
 * Handler called after the Action resumes from a redirect.
 * Only needed if you use api.redirect.sendUserTo().
 */
// exports.onContinuePostLogin = async (event, api) => {};
```

### The `event` Object

The `event` object provides context about the authentication:

```javascript
exports.onExecutePostLogin = async (event, api) => {
  // User information
  console.log(event.user.email);           // "john@example.com"
  console.log(event.user.user_id);         // "auth0|abc123"
  console.log(event.user.name);            // "John Doe"
  console.log(event.user.app_metadata);    // { roles: ["admin"] }
  console.log(event.user.user_metadata);   // { theme: "dark" }
  console.log(event.user.email_verified);  // true

  // Client (application) information
  console.log(event.client.client_id);     // "abc123def456"
  console.log(event.client.name);          // "My App"
  console.log(event.client.metadata);      // { tier: "premium" }

  // Connection information
  console.log(event.connection.id);        // "con_abc123"
  console.log(event.connection.name);      // "Username-Password-Authentication"
  console.log(event.connection.strategy);  // "auth0" or "google-oauth2" etc.

  // Transaction details
  console.log(event.transaction.protocol);        // "oidc-basic-profile"
  console.log(event.transaction.redirect_uri);    // "http://localhost:3000/callback"
  console.log(event.transaction.requested_scopes); // ["openid", "profile", "email"]
  console.log(event.transaction.locale);          // "en"

  // Tenant
  console.log(event.tenant.id);            // "my-tenant"

  // Authorization (available when RBAC is enabled)
  console.log(event.authorization?.roles);       // ["admin", "editor"]

  // Secrets (configured per Action)
  console.log(event.secrets.MY_API_KEY);   // "sk-abc123..."

  // Request info
  console.log(event.request.ip);           // "1.2.3.4"
  console.log(event.request.hostname);     // "my-tenant.us.auth0.com"
  console.log(event.request.geoip);        // { country_code: "US", ... }
  console.log(event.request.user_agent);   // "Mozilla/5.0..."
};
```

### The `api` Object

The `api` object provides methods to modify the authentication flow:

```javascript
exports.onExecutePostLogin = async (event, api) => {
  // ── Deny Access ──
  api.access.deny('Access denied: reason here');

  // ── Set Custom Claims on ID Token ──
  api.idToken.setCustomClaim('https://myapp.com/roles', ['admin']);
  api.idToken.setCustomClaim('https://myapp.com/org_id', 'org_123');

  // ── Set Custom Claims on Access Token ──
  api.accessToken.setCustomClaim('https://myapp.com/roles', ['admin']);

  // ── Update User Metadata ──
  api.user.setUserMetadata('last_login_ip', event.request.ip);
  api.user.setAppMetadata('roles', ['admin', 'editor']);

  // ── Enable MFA ──
  api.multifactor.enable('any');               // any enrolled factor
  api.multifactor.enable('any', {
    allowRememberBrowser: true,                // remember device
  });

  // ── Redirect (for step-up auth, consent, etc.) ──
  api.redirect.sendUserTo('https://myapp.com/consent', {
    query: { session_token: 'abc123' },
  });

  // ── Cache (persist data between Action calls within the same transaction) ──
  api.cache.set('key', 'value', { ttl: 60000 }); // 60 seconds
  const cached = api.cache.get('key');
};
```

### Step 3: Add Secrets

Secrets allow you to securely store API keys, tokens, and other sensitive values:

1. In the Action editor, click the **key icon** (Secrets)
2. Click **"+ Add Secret"**
3. Enter **Key** (e.g., `SLACK_WEBHOOK_URL`) and **Value**
4. Access in code via `event.secrets.SLACK_WEBHOOK_URL`

> [!warning] Secrets Are Per-Action
> Unlike Rules (which had a global `configuration` object), each Action has its own isolated secrets. This improves security but means you may need to duplicate secrets across Actions.

### Step 4: Add Dependencies

Actions support npm packages:

1. In the Action editor, click the **package icon** (Dependencies)
2. Click **"+ Add Dependency"**
3. Search for a package (e.g., `axios`, `node-fetch`, `lodash`)
4. Select a version
5. Use in code:

```javascript
const axios = require('axios');

exports.onExecutePostLogin = async (event, api) => {
  const response = await axios.get('https://api.example.com/check', {
    headers: { Authorization: `Bearer ${event.secrets.API_KEY}` },
  });

  if (response.data.blocked) {
    api.access.deny('Account is blocked');
  }
};
```

### Step 5: Test the Action

1. Click **"Test"** in the editor (the play icon)
2. Auth0 provides a mock `event` object you can customize
3. Run the test and view console output and results
4. Check if `api` methods were called correctly

### Step 6: Deploy the Action

1. Click **"Deploy"** to create a new version
2. Go to **Dashboard > Actions > Flows**
3. Select the flow (e.g., **Login**)
4. Drag your Action from the **"Add Action"** panel into the flow
5. Click **"Apply"** to activate

---

## 🔄 Action Flow and Execution Order

Actions within a flow execute **sequentially** from top to bottom (left to right in the visual editor).

```
User Authenticates
      ↓
┌─────────────────┐
│   Action 1       │  ← Runs first
│ (Enrich Token)   │
└────────┬────────┘
         ↓
┌─────────────────┐
│   Action 2       │  ← Runs second
│ (Check Blocklist)│
└────────┬────────┘
         ↓
┌─────────────────┐
│   Action 3       │  ← Runs third
│ (Force MFA)      │
└────────┬────────┘
         ↓
Token is issued → User redirected to app
```

### Important Execution Details

- **Data isolation**: Data modified in one Action's `event` object is **not** visible in subsequent Actions
- **API calls accumulate**: All `api.idToken.setCustomClaim()` calls across Actions are merged into the final token
- **Deny is final**: If any Action calls `api.access.deny()`, the pipeline stops
- **Legacy Rules run FIRST**: If you have both Rules and Actions, Rules execute before Actions
- **Ordering matters**: Drag and drop Actions in the flow editor to change order

---

## 🪝 What Are Hooks? (Deprecated)

Hooks were extensibility points for events **outside** the login pipeline. They have been replaced by Actions.

| Hook Type | Replacement Action Trigger |
|-----------|---------------------------|
| Pre User Registration | Pre User Registration Action |
| Post User Registration | Post User Registration Action |
| Post Change Password | Post Change Password Action |
| Client Credentials Exchange | Machine-to-Machine Action |
| Send Phone Message | Send Phone Message Action |

> [!danger] Hooks Are End of Life
> As of November 18, 2024, Hooks have reached their end of life. Migrate to Actions immediately.

---

## 🧩 Common Use Cases with Code Examples

### 1. Enrich Tokens with Roles and Permissions

```javascript
exports.onExecutePostLogin = async (event, api) => {
  const namespace = 'https://myapp.example.com';

  // Add roles from Auth0 RBAC
  if (event.authorization) {
    api.idToken.setCustomClaim(`${namespace}/roles`, event.authorization.roles);
    api.accessToken.setCustomClaim(`${namespace}/roles`, event.authorization.roles);
  }

  // Add custom data from app_metadata
  const tier = event.user.app_metadata?.subscription_tier || 'free';
  api.accessToken.setCustomClaim(`${namespace}/tier`, tier);
};
```

### 2. Deny Access Based on Conditions

```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Block users with unverified emails
  if (!event.user.email_verified) {
    api.access.deny('Please verify your email before logging in.');
    return;
  }

  // Block specific email domains
  const blockedDomains = ['spam.com', 'tempmail.com'];
  const domain = event.user.email?.split('@')[1];
  if (blockedDomains.includes(domain)) {
    api.access.deny('This email domain is not allowed.');
    return;
  }

  // Block outside business hours (example)
  const hour = new Date().getUTCHours();
  if (event.client.name === 'Internal Tool' && (hour < 8 || hour > 18)) {
    api.access.deny('Access is only allowed during business hours (8-18 UTC).');
  }
};
```

### 3. Call an External API

```javascript
const axios = require('axios'); // Add as dependency

exports.onExecutePostLogin = async (event, api) => {
  try {
    // Check user status in external system
    const response = await axios.post(
      event.secrets.USER_CHECK_API_URL,
      { email: event.user.email, userId: event.user.user_id },
      { headers: { 'X-API-Key': event.secrets.USER_CHECK_API_KEY }, timeout: 5000 }
    );

    if (response.data.status === 'blocked') {
      api.access.deny('Your account has been suspended.');
      return;
    }

    // Enrich token with external data
    api.accessToken.setCustomClaim(
      'https://myapp.com/external_role',
      response.data.role
    );
  } catch (error) {
    // Fail open or closed — your choice
    console.log('External API call failed:', error.message);
    // api.access.deny('Unable to verify account status.');
  }
};
```

### 4. MFA Step-Up Authentication

```javascript
exports.onExecutePostLogin = async (event, api) => {
  // Force MFA for admin users
  const roles = event.authorization?.roles || [];
  if (roles.includes('admin')) {
    api.multifactor.enable('any', { allowRememberBrowser: false });
    return;
  }

  // Force MFA for sensitive applications
  const sensitiveApps = ['ADMIN_PANEL_CLIENT_ID', 'BILLING_CLIENT_ID'];
  if (sensitiveApps.includes(event.client.client_id)) {
    api.multifactor.enable('any', { allowRememberBrowser: true });
    return;
  }

  // Force MFA for logins from new countries
  const knownCountry = event.user.user_metadata?.last_country;
  const currentCountry = event.request.geoip?.country_code;
  if (knownCountry && knownCountry !== currentCountry) {
    api.multifactor.enable('any', { allowRememberBrowser: false });
  }

  // Update last known country
  api.user.setUserMetadata('last_country', currentCountry);
};
```

### 5. Post User Registration — Welcome Email via External Service

```javascript
const axios = require('axios');

exports.onExecutePostUserRegistration = async (event) => {
  // Note: No `api` parameter for non-blocking triggers (post-registration)
  try {
    await axios.post(event.secrets.WELCOME_EMAIL_API, {
      email: event.user.email,
      name: event.user.name || event.user.email,
      signupDate: new Date().toISOString(),
    });
  } catch (error) {
    console.log('Failed to send welcome email:', error.message);
    // Non-blocking — this error does NOT affect the registration
  }
};
```

### 6. Machine-to-Machine — Enrich M2M Token

```javascript
exports.onExecuteCredentialsExchange = async (event, api) => {
  const namespace = 'https://myapp.example.com';

  // Add client metadata to access token
  if (event.client.metadata?.service_tier) {
    api.accessToken.setCustomClaim(
      `${namespace}/service_tier`,
      event.client.metadata.service_tier
    );
  }

  // Deny specific clients
  const blockedClients = ['OLD_CLIENT_ID'];
  if (blockedClients.includes(event.client.client_id)) {
    api.access.deny('This client has been decommissioned.');
  }
};
```

---

## 🔀 Migration Path from Rules to Actions

### Architecture Differences

| Aspect | Rules | Actions |
|--------|-------|---------|
| **Function params** | `(user, context, callback)` | `(event, api)` |
| **Success** | `callback(null, user, context)` | Implicit `return` |
| **Failure** | `callback(new UnauthorizedError())` | `api.access.deny()` |
| **Token claims** | `context.idToken[claim] = value` | `api.idToken.setCustomClaim(claim, value)` |
| **Metadata** | Management API required | `api.user.setAppMetadata()` |
| **MFA** | `context.multifactor = {...}` | `api.multifactor.enable()` |
| **Secrets** | `configuration.KEY` | `event.secrets.KEY` |
| **Redirect** | `context.redirect = { url }` | `api.redirect.sendUserTo(url)` |

### Data Mapping Table

| Rules (`user.*` / `context.*`) | Actions (`event.*`) |
|-------------------------------|---------------------|
| `user.email` | `event.user.email` |
| `user.user_id` | `event.user.user_id` |
| `user.app_metadata` | `event.user.app_metadata` |
| `user.user_metadata` | `event.user.user_metadata` |
| `context.clientID` | `event.client.client_id` |
| `context.clientName` | `event.client.name` |
| `context.connectionID` | `event.connection.id` |
| `context.connection` | `event.connection.name` |
| `context.connectionStrategy` | `event.connection.strategy` |
| `context.tenant` | `event.tenant.id` |
| `context.protocol` | `event.transaction.protocol` |
| `context.request.ip` | `event.request.ip` |
| `context.request.geoip` | `event.request.geoip` |

### Migration Strategy

1. **Start from the last Rule**: Migrate Rules from bottom-to-top (last in pipeline first)
2. **One Rule = One Action**: Maintain 1:1 mapping for easier debugging
3. **Test in staging**: Always test on a non-production tenant first
4. **Use metadata flags**: To prevent duplicate logic when both Rules and Actions are active
5. **Monitor**: Watch for errors in the Actions logs after migration
6. **Schedule during low traffic**: Minimize impact during the transition

### Migration Example: Rule to Action

**Original Rule:**
```javascript
function addRolesToToken(user, context, callback) {
  const namespace = 'https://myapp.com/';
  const roles = (user.app_metadata && user.app_metadata.roles) || [];

  context.idToken[namespace + 'roles'] = roles;
  context.accessToken[namespace + 'roles'] = roles;

  if (roles.includes('admin')) {
    context.multifactor = { provider: 'any', allowRememberBrowser: false };
  }

  callback(null, user, context);
}
```

**Migrated Action:**
```javascript
exports.onExecutePostLogin = async (event, api) => {
  const namespace = 'https://myapp.com';
  const roles = event.user.app_metadata?.roles || [];

  api.idToken.setCustomClaim(`${namespace}/roles`, roles);
  api.accessToken.setCustomClaim(`${namespace}/roles`, roles);

  if (roles.includes('admin')) {
    api.multifactor.enable('any', { allowRememberBrowser: false });
  }
};
```

---

## 🔗 Related Notes

- [[auth0-lesson-07-hands-on-setup]]
- [[auth0-lesson-08-web-app-integration]]
- [[auth0-lesson-10-roles-permissions-rbac]]
- [[auth0-lesson-11-security-best-practices]]
- [[auth0-lesson-12-social-enterprise-connections]]

## 📖 Sources

- [Auth0 Actions — Login Trigger](https://auth0.com/docs/customize/actions/explore-triggers/signup-and-login-triggers/login-trigger)
- [Migrate from Rules to Actions](https://auth0.com/docs/customize/actions/migrate/migrate-from-rules-to-actions)
- [Migrating Auth0 Rules to Actions (Blog)](https://auth0.com/blog/migrating-auth0-rules-to-auth0-actions/)
- [Migrating Auth0 Hooks to Actions (Blog)](https://auth0.com/blog/migrating-auth0-hooks-to-auth0-actions/)
- [Auth0 Actions Now Generally Available](https://auth0.com/blog/actions-now-generally-available/)
- [Introducing Auth0 Actions](https://auth0.com/blog/introducing-auth0-actions/)
- [Pre User Registration Trigger](https://dev.auth0.com/docs/customize/actions/explore-triggers/signup-and-login-triggers/pre-user-registration-trigger)
- [Post User Registration Trigger](https://auth0.com/docs/customize/actions/explore-triggers/signup-and-login-triggers/post-user-registration-trigger)
- [Goodbye Rules & Hooks, Hello Actions](https://medium.com/@gaschecher/auth0s-next-chapter-goodbye-rules-hooks-officially-hello-actions-c64e2a526024)

---
title: "Auth0 Lesson 2 — Core Concepts"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - status/rascunho
area: authentication
---

# 🧱 Auth0 Lesson 2 — Core Concepts

## 🎯 Context

Understanding Auth0's fundamental building blocks: Tenants, Applications, Connections, Users, and the Dashboard. These are the entities you configure and manage when working with Auth0.

## 🏠 Tenants

### What is a Tenant?

A **tenant** is the fundamental unit of isolation in Auth0. It is the container where you configure your use of Auth0, and where all Auth0 assets are defined, managed, and stored:

- Applications
- Connections
- User profiles
- Rules, Actions, and Hooks
- Branding settings
- API definitions
- Attack protection settings

> [!info] One Account, Multiple Tenants
> When you sign up for Auth0, you get one tenant. You can create additional tenants for environment isolation (dev, staging, prod) or for separating different products/business units.

### Tenant Naming and URLs

Each tenant has a unique name that forms its Auth0 domain:

```
https://{your-tenant-name}.auth0.com
```

Or with a custom domain:

```
https://auth.yourdomain.com
```

### Multi-Tenant Architecture Patterns

When building a **multi-tenant SaaS product** (where your customers each represent a "tenant" in your system), Auth0 provides several architectural approaches:

#### 1. Auth0 Organizations (Recommended)

The preferred approach for most B2B scenarios. Each of your customer organizations maps to an Auth0 Organization:

```
Your SaaS App
├── Auth0 Tenant (your-app.auth0.com)
│   ├── Organization: Acme Corp
│   │   ├── Enabled Connections: [SAML with Acme's IdP]
│   │   ├── Members: [alice@acme.com, bob@acme.com]
│   │   ├── Roles: [admin, viewer]
│   │   └── Branding: [Acme logo, colors]
│   ├── Organization: Beta Inc
│   │   ├── Enabled Connections: [Google Workspace, Database]
│   │   ├── Members: [carol@beta.com]
│   │   ├── Roles: [admin]
│   │   └── Branding: [Beta logo, colors]
│   └── ...
```

**Features:**
- Role-based access control (RBAC) per organization
- Customized login pages and email templates per organization
- Per-organization authentication connections (one org uses SAML, another uses Google)
- Organization-specific branding

#### 2. Connection-Based Tenancy

Each of your customer tenants uses a separate Auth0 connection:

- Supports varying password policies per tenant
- Allows different authentication methods (username/password vs. enterprise IdP)
- **Limitation**: Lock (classic login widget) supports a maximum of 50 database connections per application

#### 3. Application-Based Tenancy

Each customer tenant maps to a unique Auth0 application:

- Individual configuration per tenant (callbacks, allowed origins, etc.)
- Application must track tenant associations
- More management overhead

#### 4. Auth0 Tenant-Based Tenancy

Separate Auth0 tenants for each customer:

- Maximum isolation — each customer has their own Auth0 tenant
- Allows restricted Auth0 Dashboard access per customer
- **Trade-off**: Requires individual configuration management (Branding, Actions, Attack Protection) across all tenant instances
- Most expensive and complex to manage

#### 5. User Profile Storage (app_metadata)

Store tenant information in the `app_metadata` field of the user profile:

- Uniform login configuration across all tenants
- Application retrieves tenant ID post-authentication
- Simplest approach but least isolated

### Environment Strategy (Dev / Staging / Prod)

Best practice is to create **separate Auth0 tenants** for each environment:

```
my-app-dev.auth0.com      → Development
my-app-staging.auth0.com   → Staging
my-app-prod.auth0.com      → Production
```

> [!warning] Configuration Drift
> Each tenant is independently configured. Use **Auth0 Deploy CLI** or **Terraform Auth0 Provider** to keep configurations synchronized across environments. Auth0 also offers "Dry Run" mode for the Deploy CLI to preview changes before applying.

## 📱 Applications

### What is an Application in Auth0?

An **Application** in Auth0 represents the software that will interact with Auth0 for authentication. When you register an application, Auth0 assigns it:

- **Client ID** — A public identifier for the application (safe to expose in frontend code)
- **Client Secret** — A private key used for server-side authentication (must be kept secret)

### Application Types

| Type | Description | Example | Can Store Secrets? | Recommended Flow |
|------|-------------|---------|-------------------|-----------------|
| **Regular Web Application** | Server-rendered apps with a backend | Express.js, ASP.NET, Rails, Django | Yes (confidential) | Authorization Code |
| **Single Page Application (SPA)** | Client-side JavaScript apps | React, Angular, Vue | No (public) | Authorization Code + PKCE |
| **Native Application** | Mobile or desktop apps | iOS, Android, Electron | No (public) | Authorization Code + PKCE |
| **Machine-to-Machine (M2M)** | Backend services, CLIs, daemons, IoT | Cron jobs, microservices, APIs | Yes (confidential) | Client Credentials |

### Confidential vs. Public Clients

This distinction comes from the OAuth 2.0 spec:

- **Confidential clients** — Can securely store credentials (server-side apps). They have a client secret.
- **Public clients** — Cannot securely store credentials (browser apps, mobile apps). They must **never** contain a client secret.

> [!danger] Security Rule
> Mobile and browser-based applications should **never** contain client secrets. The secret would be extractable from the app binary or browser source code.

### Key Application Settings

| Setting | Purpose |
|---------|---------|
| **Allowed Callback URLs** | URLs Auth0 can redirect to after authentication |
| **Allowed Logout URLs** | URLs Auth0 can redirect to after logout |
| **Allowed Web Origins** | Origins for cross-origin authentication and silent token renewal |
| **Allowed Origins (CORS)** | Origins for Cross-Origin Resource Sharing |
| **Token Endpoint Auth Method** | How the app authenticates to the token endpoint (Post, Basic, None) |
| **Grant Types** | Which OAuth grant types the app can use |
| **Connections** | Which identity connections are enabled for this app |

### Application Configuration Example

```javascript
// Example: Configuring Auth0 SPA SDK
import { createAuth0Client } from '@auth0/auth0-spa-js';

const auth0 = await createAuth0Client({
  domain: 'my-app.auth0.com',        // Tenant domain
  clientId: 'abc123def456',           // Application Client ID
  authorizationParams: {
    redirect_uri: 'https://myapp.com/callback',  // Allowed Callback URL
    audience: 'https://api.myapp.com',            // API identifier
    scope: 'openid profile email read:data'       // Requested scopes
  }
});
```

## 🔌 Connections

### What is a Connection?

A **Connection** is a source of users — the method by which users authenticate. Each connection defines where and how user credentials are verified.

### Connection Types

#### 1. Database Connections

Auth0 stores user credentials directly in its own database:

- Fully managed by Auth0
- Customizable password policies (length, complexity, history)
- Support for username or email-based login
- Custom database connections can proxy to your own user store via scripts
- Import users from external databases on first login (lazy migration)

```javascript
// Example: Custom Database - Login Script (Node.js)
async function login(email, password, callback) {
  const user = await myLegacyDb.findUserByEmail(email);
  if (!user) return callback(new WrongUsernameOrPasswordError(email));

  const isValid = await bcrypt.compare(password, user.passwordHash);
  if (!isValid) return callback(new WrongUsernameOrPasswordError(email));

  return callback(null, {
    user_id: user.id,
    email: user.email,
    name: user.name
  });
}
```

#### 2. Social Connections

Third-party social identity providers:

- **30+ providers supported**: Google, Facebook, GitHub, Apple, Twitter/X, LinkedIn, Microsoft, Amazon, etc.
- Each requires creating an OAuth app on the provider's platform and configuring the client ID/secret in Auth0
- Auth0 provides "Dev Keys" for quick testing (not for production)
- Returns profile information from the social provider (name, email, avatar, etc.)

#### 3. Enterprise Connections

For organizations with existing identity infrastructure:

| Protocol | Providers |
|----------|-----------|
| **SAML** | Any SAML 2.0 IdP (Okta, Azure AD, ADFS, Ping) |
| **OIDC** | Any OpenID Connect provider |
| **LDAP/AD** | Active Directory via AD/LDAP Connector |
| **Azure AD** | Direct integration with Microsoft Azure Active Directory |
| **Google Workspace** | Google as enterprise IdP |
| **ADFS** | Active Directory Federation Services |

#### 4. Passwordless Connections

Authentication without traditional passwords:

| Method | How It Works |
|--------|-------------|
| **Email Magic Link** | User receives a link via email; clicking it authenticates them |
| **Email OTP** | User receives a one-time code via email |
| **SMS OTP** | User receives a one-time code via SMS |
| **WebAuthn/FIDO2** | Biometric authentication (fingerprint, face) or hardware keys |

> [!note] Passwordless Identity
> Passwordless connections create a **distinct identity** in Auth0. A user with `alice@example.com` via database connection is a **different user** than `alice@example.com` via passwordless. Use **Account Linking** to merge them.

### Connection-to-Application Mapping

Connections are enabled **per application**. You can have multiple connections but only enable specific ones for each app:

```
Web App (SPA)
├── Enabled: Database, Google, GitHub
└── Disabled: SAML (not needed for consumer app)

Admin Portal (Regular Web)
├── Enabled: Database, Azure AD (SAML)
└── Disabled: Google, GitHub

Backend Service (M2M)
└── No connections (uses Client Credentials)
```

## 👤 Users

### User Profiles

Every authenticated user has a **User Profile** in Auth0 containing:

- **user_id** — Unique identifier (format: `{connection}|{id}`, e.g., `auth0|507f1f77bcf86cd799439011`)
- **email** — Email address
- **name** — Display name
- **picture** — Avatar URL
- **identities** — Array of linked provider identities
- **created_at** / **updated_at** — Timestamps
- **last_login** — Last authentication timestamp
- **logins_count** — Total login count

### Metadata: user_metadata vs. app_metadata

This is a critical distinction:

| Aspect | `user_metadata` | `app_metadata` |
|--------|-----------------|----------------|
| **Purpose** | User preferences that don't affect core functionality | Data that affects app functionality and access |
| **Who can read?** | User (via Management API), Application | Application only |
| **Who can write?** | User (if you expose a form), Application | Application only (via Management API) |
| **Examples** | Language preference, theme, timezone, display name | Roles, plan type, subscription status, feature flags |
| **Security** | Not a secure data store | Used for authorization decisions |
| **Size limit** | 10 fields, 500 chars max (via signup endpoint) | No specific documented limit |
| **Max payload** | Part of total 100KB user profile limit | Part of total 100KB user profile limit |

```javascript
// Example: User profile with metadata
{
  "user_id": "auth0|507f1f77bcf86cd799439011",
  "email": "alice@example.com",
  "name": "Alice Smith",
  "user_metadata": {
    "language": "pt-BR",
    "theme": "dark",
    "timezone": "America/Sao_Paulo"
  },
  "app_metadata": {
    "plan": "enterprise",
    "roles": ["admin", "billing"],
    "organization_id": "org_abc123",
    "feature_flags": {
      "beta_dashboard": true
    }
  }
}
```

### Updating Metadata

```javascript
// Updating user_metadata (can be done by authenticated user)
const response = await fetch(
  `https://${domain}/api/v2/users/${userId}`,
  {
    method: 'PATCH',
    headers: {
      'Authorization': `Bearer ${managementApiToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      user_metadata: { language: 'en' }
    })
  }
);

// Updating app_metadata (backend only, never from client)
const response = await fetch(
  `https://${domain}/api/v2/users/${userId}`,
  {
    method: 'PATCH',
    headers: {
      'Authorization': `Bearer ${managementApiToken}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      app_metadata: { plan: 'premium' }
    })
  }
);
```

### User Management Operations

| Operation | Dashboard | Management API | Actions |
|-----------|-----------|---------------|---------|
| Create user | Yes | `POST /api/v2/users` | Yes |
| Read user | Yes | `GET /api/v2/users/{id}` | Yes |
| Update user | Yes | `PATCH /api/v2/users/{id}` | Yes |
| Delete user | Yes | `DELETE /api/v2/users/{id}` | N/A |
| Search users | Yes | `GET /api/v2/users?q=...` | N/A |
| Block/Unblock | Yes | `PATCH /api/v2/users/{id}` | Yes |
| Link accounts | No | `POST /api/v2/users/{id}/identities` | Yes |
| Assign roles | Yes | `POST /api/v2/users/{id}/roles` | Yes |

## 🖥️ The Auth0 Dashboard

The Auth0 Dashboard is the web-based admin interface for managing your Auth0 tenant. Key sections:

### Dashboard Sections

| Section | Purpose |
|---------|---------|
| **Getting Started** | Quickstart guides and setup wizard |
| **Applications** | Register and configure applications |
| **APIs** | Define and manage API definitions (audiences) |
| **Authentication** | Configure connections (Database, Social, Enterprise, Passwordless) |
| **User Management** | View/search users, manage roles and permissions |
| **Organizations** | Configure B2B multi-tenancy |
| **Actions** | Create and manage serverless functions for auth events |
| **Branding** | Customize Universal Login, email templates, custom domains |
| **Security** | Attack protection, MFA policies, suspicious activity |
| **Monitoring** | Logs, log streaming, tenant health |
| **Marketplace** | Browse and install Auth0 integrations |
| **Settings** | Tenant-level settings, signing keys, advanced configuration |

### Management API

Everything available in the Dashboard is also available via the **Management API**, enabling Infrastructure as Code:

```bash
# Example: List all applications via Management API
curl --request GET \
  --url 'https://my-tenant.auth0.com/api/v2/clients' \
  --header 'Authorization: Bearer {MANAGEMENT_API_TOKEN}'
```

### Auth0 CLI

Auth0 also provides a CLI tool for managing tenants:

```bash
# Install
brew install auth0/auth0-cli/auth0

# Login
auth0 login

# List applications
auth0 apps list

# Create an application
auth0 apps create --name "My API" --type m2m

# View logs
auth0 logs tail
```

## 🔗 Sources

- [Applications in Auth0 — Auth0 Docs](https://auth0.com/docs/get-started/applications)
- [Multi-Tenant Applications Best Practices — Auth0 Docs](https://auth0.com/docs/get-started/auth0-overview/create-tenants/multi-tenant-apps-best-practices)
- [Understand How Metadata Works in User Profiles — Auth0 Docs](https://auth0.com/docs/manage-users/user-accounts/metadata)
- [Using Auth0 for B2B Multi/Single-Tenant SaaS Solutions](https://auth0.com/blog/using-auth0-for-b2b-multi-and-single-tenant-saas-solutions/)
- [Implementing Multitenancy in Auth0 Using Organizations](https://dev.to/akarshan/implementing-multitenancy-in-auth0-using-organizations-a-complete-guide-433c)
- [Passwordless Authentication — Auth0 Docs](https://auth0.com/docs/authenticate/passwordless)

## 📝 Related Notes

- [[auth0-lesson-01-what-is-auth0]]
- [[auth0-lesson-03-authentication-protocols]]
- [[auth0-lesson-04-auth-flows]]
- [[auth0-lesson-05-tokens]]
- [[auth0-lesson-06-jwt-deep-dive]]

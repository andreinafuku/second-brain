---
title: "Auth0 Lesson 10 — Roles & Permissions (RBAC)"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - auth0
  - identity
  - status/rascunho
area: arquitetura
---

# 🔐 Auth0 Lesson 10 — Roles & Permissions (RBAC)

## 🎯 Overview

This lesson covers how Auth0 implements Role-Based Access Control (RBAC), how to configure roles and permissions, embed them in tokens, enforce them in your API, and handle multi-tenant SaaS scenarios with Organizations.

---

## 🔑 Authentication vs Authorization

| Aspect | Authentication (AuthN) | Authorization (AuthZ) |
|--------|----------------------|----------------------|
| **Question** | "Who are you?" | "What can you do?" |
| **Mechanism** | Login (credentials, SSO, MFA) | Permissions, roles, policies |
| **Protocol** | OIDC / SAML | OAuth 2.0 scopes, custom claims |
| **Token** | ID Token | Access Token |
| **Result** | User identity confirmed | Access granted or denied |

Auth0 handles **authentication** natively. For **authorization**, Auth0 provides:

1. **RBAC** (built-in roles and permissions)
2. **Actions** (custom authorization logic)
3. **Auth0 Fine-Grained Authorization (FGA)** — a separate product for relationship-based access control (like Google Zanzibar / OpenFGA)

---

## 🏗️ Auth0's RBAC Model

Auth0's RBAC model consists of three components:

```
Permissions (defined per API)
    ↓
Roles (collections of permissions)
    ↓
Users (assigned one or more roles)
```

**Key principle**: Auth0 uses an **additive model** — if a user has multiple roles, they receive the **union** of all permissions from all roles.

Example:
- Role `editor` has permissions: `read:articles`, `write:articles`
- Role `moderator` has permissions: `read:articles`, `delete:comments`
- User with both roles gets: `read:articles`, `write:articles`, `delete:comments`

---

## ⚙️ Creating Roles and Permissions in Auth0

### Step 1: Define Permissions on Your API

1. Go to **Dashboard > Applications > APIs**
2. Select your API (or create one)
3. Go to the **Permissions** tab
4. Add permissions:

| Permission | Description |
|------------|-------------|
| `read:users` | Read user profiles |
| `write:users` | Create/update users |
| `delete:users` | Delete users |
| `read:reports` | View reports |
| `admin:all` | Full administrative access |

> [!tip] Permission Naming Convention
> Use the format `action:resource` — e.g., `read:articles`, `write:orders`, `delete:users`. This is consistent, scannable, and maps well to API endpoints.

### Step 2: Enable RBAC on Your API

1. On the same API page, go to the **Settings** tab
2. Scroll to **RBAC Settings**
3. Enable:
   - **Enable RBAC**: Turns on role-based access control
   - **Add Permissions in the Access Token**: Includes the `permissions` claim in the access token

> [!warning] Enable "Add Permissions in the Access Token"
> Without this toggle, the access token will **not** contain the `permissions` array. Your API will have no way to check permissions from the token alone.

### Step 3: Create Roles

1. Go to **Dashboard > User Management > Roles**
2. Click **"+ Create Role"**
3. Enter:
   - **Name**: `admin`, `editor`, `viewer`
   - **Description**: Clear description of the role's purpose
4. Go to the **Permissions** tab of the role
5. Click **"Add Permissions"**
6. Select the API and check the permissions for this role

Example roles:

```
Role: viewer
  Permissions: read:users, read:reports

Role: editor
  Permissions: read:users, read:reports, write:users

Role: admin
  Permissions: read:users, read:reports, write:users, delete:users, admin:all
```

### Step 4: Assign Roles to Users

#### Via Dashboard

1. Go to **Dashboard > User Management > Users**
2. Select a user
3. Go to the **Roles** tab
4. Click **"Assign Roles"**
5. Select the roles to assign

#### Via Management API

```javascript
// Using the Auth0 Management API client
const ManagementClient = require('auth0').ManagementClient;

const management = new ManagementClient({
  domain: 'YOUR_AUTH0_DOMAIN',
  clientId: 'M2M_CLIENT_ID',
  clientSecret: 'M2M_CLIENT_SECRET',
});

// Assign roles to a user
await management.users.assignRoles(
  { id: 'auth0|user123' },
  { roles: ['rol_admin123', 'rol_editor456'] }
);

// Remove roles from a user
await management.users.removeRoles(
  { id: 'auth0|user123' },
  { roles: ['rol_editor456'] }
);

// Get user's roles
const roles = await management.users.getRoles({ id: 'auth0|user123' });
console.log(roles.data); // [{ id: 'rol_admin123', name: 'admin', ... }]

// Get user's permissions (including those from roles)
const permissions = await management.users.getPermissions({ id: 'auth0|user123' });
console.log(permissions.data); // [{ permission_name: 'read:users', ... }]
```

#### Via Auth0 CLI

```bash
# Assign a role
auth0 users roles assign USER_ID --roles ROLE_ID

# List user's roles
auth0 users roles show USER_ID
```

---

## 🎫 Adding Permissions to Access Tokens

Once RBAC is enabled and "Add Permissions in the Access Token" is toggled on, the access token will contain:

```json
{
  "iss": "https://your-tenant.us.auth0.com/",
  "sub": "auth0|user123",
  "aud": "https://my-api.example.com",
  "iat": 1681234567,
  "exp": 1681320967,
  "scope": "openid profile email",
  "permissions": [
    "read:users",
    "write:users",
    "read:reports"
  ]
}
```

### Adding Roles to Tokens via Actions

By default, Auth0 does **not** include roles in tokens — only permissions. To include roles, use a Post Login Action:

```javascript
exports.onExecutePostLogin = async (event, api) => {
  const namespace = 'https://myapp.example.com';

  if (event.authorization) {
    // Add roles to both ID and access tokens
    api.idToken.setCustomClaim(`${namespace}/roles`, event.authorization.roles);
    api.accessToken.setCustomClaim(`${namespace}/roles`, event.authorization.roles);
  }
};
```

This produces:

```json
{
  "https://myapp.example.com/roles": ["admin", "editor"],
  "permissions": ["read:users", "write:users", "read:reports"]
}
```

> [!warning] Namespace Requirement
> Custom claims MUST use a namespaced URI format (e.g., `https://myapp.example.com/roles`). Claims without a namespace are silently dropped by Auth0 to avoid conflicts with registered OIDC claims.

---

## ✅ Checking Permissions in Your API

### Using express-oauth2-jwt-bearer

```javascript
const express = require('express');
const { auth, requiredScopes, claimCheck } = require('express-oauth2-jwt-bearer');

const app = express();

const checkJwt = auth({
  audience: 'https://my-api.example.com',
  issuerBaseURL: 'https://YOUR_AUTH0_DOMAIN',
  tokenSigningAlg: 'RS256',
});

// ── Method 1: Check scopes ──
app.get('/api/users', checkJwt, requiredScopes('read:users'), (req, res) => {
  res.json({ users: [] });
});

// ── Method 2: Check permissions array ──
const checkPermissions = (...requiredPermissions) => {
  return (req, res, next) => {
    const permissions = req.auth.payload.permissions || [];
    const hasAll = requiredPermissions.every(p => permissions.includes(p));
    if (!hasAll) {
      return res.status(403).json({
        error: 'Forbidden',
        message: `Missing permissions: ${requiredPermissions.filter(p => !permissions.includes(p)).join(', ')}`,
      });
    }
    next();
  };
};

app.get('/api/users', checkJwt, checkPermissions('read:users'), (req, res) => {
  res.json({ users: [] });
});

app.post('/api/users', checkJwt, checkPermissions('write:users'), (req, res) => {
  res.json({ message: 'User created' });
});

app.delete('/api/users/:id', checkJwt, checkPermissions('delete:users'), (req, res) => {
  res.json({ message: 'User deleted' });
});

// ── Method 3: Check roles (from custom claim) ──
const checkRoles = (...requiredRoles) => {
  return (req, res, next) => {
    const roles = req.auth.payload['https://myapp.example.com/roles'] || [];
    const hasRole = requiredRoles.some(r => roles.includes(r));
    if (!hasRole) {
      return res.status(403).json({
        error: 'Forbidden',
        message: `Required role: ${requiredRoles.join(' or ')}`,
      });
    }
    next();
  };
};

app.get('/api/admin', checkJwt, checkRoles('admin'), (req, res) => {
  res.json({ message: 'Admin area' });
});

// ── Method 4: Using claimCheck ──
app.get('/api/premium',
  checkJwt,
  claimCheck((claims) => {
    return claims.permissions?.includes('read:premium') &&
           claims['https://myapp.example.com/roles']?.includes('premium');
  }),
  (req, res) => {
    res.json({ message: 'Premium content' });
  }
);

app.listen(3001);
```

### Checking Permissions in React (Frontend)

```tsx
import { useAuth0 } from '@auth0/auth0-react';

const AdminPanel = () => {
  const { user, isAuthenticated } = useAuth0();

  // Roles are in the custom claim
  const roles = user?.['https://myapp.example.com/roles'] || [];
  const isAdmin = roles.includes('admin');

  if (!isAuthenticated || !isAdmin) {
    return <p>You do not have access to this page.</p>;
  }

  return <div>Admin Panel Content</div>;
};
```

> [!danger] Never Trust the Frontend Alone
> Frontend role/permission checks are for **UX purposes only** (hiding/showing UI elements). Always enforce authorization in your **API/backend**. A malicious user can bypass any frontend check.

---

## 🏢 Auth0 Organizations (Multi-Tenant SaaS)

Organizations are Auth0's feature for **B2B SaaS** applications where your customers are companies (organizations) with their own users, roles, and identity providers.

### Key Concepts

- Each **Organization** represents a customer/tenant in your SaaS
- Users can belong to **multiple Organizations** with different roles in each
- Each Organization can have its own **enabled connections** (e.g., Company A uses Azure AD, Company B uses Google)
- Organizations map 1:1 with your SaaS tenants

### Creating an Organization

#### Via Dashboard

1. Go to **Dashboard > Organizations**
2. Click **"+ Create Organization"**
3. Enter:
   - **Name**: Machine-friendly identifier (e.g., `acme-corp`)
   - **Display Name**: Human-friendly name (e.g., "Acme Corporation")
   - **Logo URL**: Organization logo (optional)
   - **Metadata**: Key-value pairs (e.g., `{ "plan": "enterprise", "region": "us-east" }`)

#### Via Management API

```javascript
// Create an organization
const org = await management.organizations.create({
  name: 'acme-corp',
  display_name: 'Acme Corporation',
  branding: {
    logo_url: 'https://acme.com/logo.png',
    colors: { primary: '#FF5733', page_background: '#FFFFFF' },
  },
  metadata: { plan: 'enterprise', region: 'us-east' },
});

// Enable a connection for the organization
await management.organizations.addEnabledConnection(
  { id: org.data.id },
  {
    connection_id: 'con_azure_ad_abc123',
    assign_membership_on_login: true, // Auto-add users who log in
  }
);

// Add a member
await management.organizations.addMembers(
  { id: org.data.id },
  { members: ['auth0|user123'] }
);

// Assign roles to a member within the organization
await management.organizations.addMemberRoles(
  { id: org.data.id, user_id: 'auth0|user123' },
  { roles: ['rol_org_admin'] }
);
```

### Inviting Members

```javascript
// Send an invitation
const invitation = await management.organizations.createInvitation(
  { id: 'org_id_abc123' },
  {
    inviter: { name: 'Admin User' },
    invitee: { email: 'newuser@acme.com' },
    client_id: 'YOUR_CLIENT_ID',
    connection_id: 'con_db_123',        // optional: specify connection
    roles: ['rol_org_member'],           // optional: assign roles on accept
    send_invitation_email: true,         // Auth0 sends the email
    ttl_sec: 604800,                     // 7 days to accept
  }
);
```

### Using Organizations in Your App

```tsx
// React — login with organization
<Auth0Provider
  domain={import.meta.env.VITE_AUTH0_DOMAIN}
  clientId={import.meta.env.VITE_AUTH0_CLIENT_ID}
  authorizationParams={{
    redirect_uri: window.location.origin,
    organization: 'org_acmeCorp123',  // specific org
    // OR let the user pick during login
  }}
>
```

```javascript
// Express — validate org in token
app.get('/api/org-data', checkJwt, (req, res) => {
  const orgId = req.auth.payload.org_id;
  if (!orgId) {
    return res.status(403).json({ error: 'Organization context required' });
  }
  // Fetch data scoped to the organization
  res.json({ org_id: orgId, data: '...' });
});
```

### Organization Token Claims

When a user logs in with an Organization context, the access token includes:

```json
{
  "org_id": "org_acmeCorp123",
  "org_name": "acme-corp",
  "permissions": ["read:data", "write:data"],
  "sub": "auth0|user123"
}
```

> [!tip] Organization-Scoped Roles
> A user can be an `admin` in Organization A but a `viewer` in Organization B. Roles assigned via Organizations are **scoped** to that organization.

---

## 🎯 Fine-Grained Authorization Patterns

For scenarios where RBAC is not granular enough, consider these patterns:

### 1. Attribute-Based Access Control (ABAC)

Check attributes beyond roles — department, location, time of day, resource owner:

```javascript
// In an Action or API middleware
exports.onExecutePostLogin = async (event, api) => {
  const dept = event.user.app_metadata?.department;
  const geo = event.request.geoip?.country_code;

  api.accessToken.setCustomClaim('https://myapp.com/department', dept);
  api.accessToken.setCustomClaim('https://myapp.com/country', geo);
};

// In API
app.get('/api/sensitive-data', checkJwt, (req, res) => {
  const dept = req.auth.payload['https://myapp.com/department'];
  const country = req.auth.payload['https://myapp.com/country'];

  if (dept !== 'engineering' || country !== 'US') {
    return res.status(403).json({ error: 'Access restricted' });
  }
  res.json({ data: 'sensitive' });
});
```

### 2. Resource-Based Access Control

Check if the user owns or has access to a specific resource:

```javascript
app.get('/api/documents/:id', checkJwt, async (req, res) => {
  const userId = req.auth.payload.sub;
  const document = await db.documents.findById(req.params.id);

  if (!document) return res.status(404).json({ error: 'Not found' });

  // Check ownership or shared access
  const hasAccess = document.owner_id === userId ||
                    document.shared_with.includes(userId);

  if (!hasAccess) return res.status(403).json({ error: 'Forbidden' });

  res.json(document);
});
```

### 3. Auth0 Fine-Grained Authorization (FGA)

For complex scenarios (Google Docs-style sharing, hierarchical permissions), Auth0 FGA (based on OpenFGA / Google Zanzibar):

```javascript
// Check: can user X view document Y?
const { allowed } = await fgaClient.check({
  user: 'user:auth0|user123',
  relation: 'viewer',
  object: 'document:readme',
});

if (!allowed) {
  return res.status(403).json({ error: 'Forbidden' });
}
```

> [!info] Auth0 FGA
> Auth0 FGA is a **separate product** from Auth0's core RBAC. It handles relationship-based authorization (ReBAC) for complex permission models. It's inspired by Google's Zanzibar paper and is the same technology behind OpenFGA.

---

## 📊 RBAC Decision Flowchart

```
Is the request authenticated?
    ├── No → 401 Unauthorized
    └── Yes → Does the token contain required permissions?
                  ├── No → 403 Forbidden
                  └── Yes → Does the user have access to THIS resource?
                                ├── No → 403 Forbidden
                                └── Yes → 200 OK (serve the resource)
```

---

## 🔗 Related Notes

- [[auth0-lesson-07-hands-on-setup]]
- [[auth0-lesson-08-web-app-integration]]
- [[auth0-lesson-09-rules-actions-hooks]]
- [[auth0-lesson-11-security-best-practices]]
- [[auth0-lesson-12-social-enterprise-connections]]

## 📖 Sources

- [Auth0 RBAC Documentation](https://dev.auth0.com/docs/manage-users/access-control/rbac)
- [Auth0 Organizations Documentation](https://auth0.com/docs/manage-users/organizations)
- [Invite Organization Members](https://dev.auth0.com/docs/manage-users/organizations/configure-organizations/invite-members)
- [How to Choose the Right Authorization Model](https://auth0.com/blog/how-to-choose-the-right-authorization-model-for-your-multi-tenant-saas-application/)
- [Using Auth0 for B2B Multi/Single-Tenant SaaS](https://auth0.com/blog/using-auth0-for-b2b-multi-and-single-tenant-saas-solutions/)
- [Role Management with Auth0 Organizations for B2B SaaS](https://auth0.com/blog/role-management-auth0-organizations-b2b-saas/)
- [Understanding RBAC and Organizations with Auth0](https://www.enhisecure.com/isecureblog/2026/03/13/understanding-rbac-and-organizations-with-auth0/)
- [Implementing RBAC with Auth0 FGA and FastAPI](https://auth0.com/blog/implementing-rbac-fastapi-auth0-fga/)
- [B2B SaaS Identity Challenges: Granular Access Control](https://auth0.com/blog/b2b-saas-identity-challenges-granular-access-control/)

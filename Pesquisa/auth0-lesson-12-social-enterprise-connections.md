---
title: "Auth0 Lesson 12 — Social & Enterprise Connections"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - auth0
  - identity
  - status/rascunho
area: arquitetura
---

# 🌍 Auth0 Lesson 12 — Social & Enterprise Connections

## 🎯 Overview

This lesson covers the full spectrum of identity connections in Auth0: social providers (Google, GitHub, Facebook, Apple, LinkedIn), enterprise protocols (SAML, Azure AD, Active Directory/LDAP, Google Workspace), passwordless authentication, account linking, and home realm discovery.

---

## 🔵 Social Connections

Social connections allow users to authenticate using their existing accounts from popular platforms. Auth0 supports 30+ social identity providers out of the box.

### Supported Social Providers

| Provider | Strategy Name | Common Scopes |
|----------|--------------|---------------|
| **Google** | `google-oauth2` | `email`, `profile`, `openid` |
| **GitHub** | `github` | `user:email`, `read:user`, `read:org` |
| **Facebook** | `facebook` | `email`, `public_profile` |
| **Apple** | `apple` | `email`, `name` |
| **LinkedIn** | `linkedin` | `openid`, `profile`, `email` |
| **Microsoft** | `windowslive` | `openid`, `profile`, `email` |
| **Twitter/X** | `twitter` | Read-only profile |
| **Slack** | `slack` | `identity.basic`, `identity.email` |
| **Discord** | `discord` | `identify`, `email` |

### Setting Up a Social Connection (Step by Step)

The process is similar across providers. Here is the general flow using **Google** as an example:

#### Step 1: Create OAuth Credentials at the Provider

**Google:**
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a project (or select existing)
3. Navigate to **APIs & Services > OAuth Consent Screen**
   - User Type: **External** (or Internal for Google Workspace orgs)
   - Fill in app name, support email, developer email
   - Add scopes: `openid`, `email`, `profile`
4. Go to **Credentials > Create Credentials > OAuth Client ID**
   - Application type: **Web application**
   - Authorized redirect URIs: `https://YOUR_AUTH0_DOMAIN/login/callback`
5. Copy **Client ID** and **Client Secret**

**GitHub:**
1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Click **"New OAuth App"**
3. Fill in:
   - Application name
   - Homepage URL: `https://YOUR_AUTH0_DOMAIN`
   - Authorization callback URL: `https://YOUR_AUTH0_DOMAIN/login/callback`
4. Copy **Client ID** and **Client Secret**

**Facebook:**
1. Go to [Facebook Developers](https://developers.facebook.com/)
2. Create a new app (Consumer type)
3. Add **Facebook Login** product
4. Set Valid OAuth Redirect URIs: `https://YOUR_AUTH0_DOMAIN/login/callback`
5. Copy **App ID** and **App Secret**

**Apple:**
1. Go to [Apple Developer Portal](https://developer.apple.com/)
2. Register an **App ID** with "Sign in with Apple" capability
3. Create a **Services ID** (this is your Client ID)
4. Configure **Web Authentication**: Domain + Return URL (`https://YOUR_AUTH0_DOMAIN/login/callback`)
5. Create a **Key** for Sign in with Apple → download the `.p8` key file
6. You need: **Services ID** (Client ID), **Team ID**, **Key ID**, and **Key File**

**LinkedIn:**
1. Go to [LinkedIn Developers](https://www.linkedin.com/developers/)
2. Create a new app
3. Under **Auth** tab, add redirect URL: `https://YOUR_AUTH0_DOMAIN/login/callback`
4. Request products: "Sign In with LinkedIn using OpenID Connect"
5. Copy **Client ID** and **Client Secret**

#### Step 2: Configure in Auth0

1. Go to **Dashboard > Authentication > Social**
2. Click **"+ Create Connection"**
3. Select the provider
4. Enter your **Client ID** and **Client Secret** (or keys for Apple)
5. Configure **scopes** (permissions to request from the provider)
6. Under **Applications** tab, enable for your apps
7. Click **Save**

#### Step 3: Test the Connection

1. On the connection page, click **"Try Connection"**
2. You are redirected to the provider's login page
3. After authenticating, you see the returned profile data

### Scopes and Permissions per Provider

Scopes determine what data your app can access from the social provider.

```
Google:
  - email           → User's email address
  - profile         → Name, picture, locale
  - openid          → OpenID Connect identity
  - calendar.readonly → Read Google Calendar (requires verification)
  - drive.readonly  → Read Google Drive files (requires verification)

GitHub:
  - user:email      → Email addresses
  - read:user       → Profile info
  - read:org        → Organization membership
  - repo            → Repository access (sensitive!)

Facebook:
  - email           → Email address
  - public_profile  → Name, picture, age range
  - user_birthday   → Birthday (requires App Review)
  - user_location   → Location (requires App Review)

Apple:
  - email           → Email (may be a private relay address)
  - name            → Full name (only on first login!)
```

> [!warning] Apple Sign-In Specifics
> Apple only returns the user's **name** on the **first login**. If you don't capture and store it at that point, you cannot retrieve it later. Apple also offers **private relay email** (`abc123@privaterelay.appleid.com`), hiding the user's real email.

> [!warning] Provider Verification
> Many providers (Google, Facebook) require **app verification** before you can request sensitive scopes or go to production. Unverified apps are limited (e.g., Google limits to 100 users). Start the verification process early.

### Auth0 Development Keys

For quick development, Auth0 provides **built-in development keys** for major social providers:

- No need to create credentials at Google/GitHub/Facebook
- Consent screen shows "Powered by Auth0" branding
- Limited functionality — **not for production**
- To switch to your own keys: Edit the connection and enter your credentials

---

## 🔗 Account Linking

When a user logs in via Google and later via email/password, Auth0 creates **two separate user profiles**. Account linking merges them.

### Why Account Linking?

- Same user, different providers → different `user_id` values
- `google-oauth2|123` and `auth0|456` are different identities
- Account linking connects them under a single primary identity

### Automatic Account Linking

Auth0 provides a **pre-built Action** for automatic linking:

1. Go to **Dashboard > Actions > Library > Marketplace**
2. Search for **"Account Linking"**
3. Install the Action
4. Configure:
   - Which connections to link
   - Whether to prompt the user or auto-link
5. Add to the **Login** flow

### Manual Account Linking via Management API

```javascript
// Link two accounts
await management.users.link(
  { id: 'auth0|primary_user_id' },  // Primary account
  {
    provider: 'google-oauth2',        // Secondary account provider
    user_id: 'google-oauth2|secondary_user_id',
  }
);

// Unlink accounts
await management.users.unlink({
  id: 'auth0|primary_user_id',
  provider: 'google-oauth2',
  user_id: 'secondary_user_id',
});

// Get linked identities
const user = await management.users.get({ id: 'auth0|primary_user_id' });
console.log(user.data.identities);
// [
//   { provider: 'auth0', user_id: 'primary_user_id', connection: 'Username-Password-Authentication' },
//   { provider: 'google-oauth2', user_id: '123456789', connection: 'google-oauth2' }
// ]
```

### Account Linking via Action

```javascript
const { ManagementClient } = require('auth0');

exports.onExecutePostLogin = async (event, api) => {
  // Skip if this is already a linked account
  if (event.user.identities && event.user.identities.length > 1) return;

  const management = new ManagementClient({
    domain: event.secrets.AUTH0_DOMAIN,
    clientId: event.secrets.AUTH0_M2M_CLIENT_ID,
    clientSecret: event.secrets.AUTH0_M2M_CLIENT_SECRET,
  });

  // Find users with the same email
  const users = await management.usersByEmail.getByEmail({
    email: event.user.email,
  });

  // If there are other accounts with the same verified email, link them
  for (const existingUser of users) {
    if (existingUser.user_id === event.user.user_id) continue;
    if (!existingUser.email_verified || !event.user.email_verified) continue;

    // Link: existing user becomes the primary
    await management.users.link(
      { id: existingUser.user_id },
      {
        provider: event.connection.strategy,
        user_id: event.user.user_id,
      }
    );
    break;
  }
};
```

> [!warning] Account Linking Security
> Only link accounts when **both emails are verified**. Linking unverified accounts is a security risk — an attacker could create an account with your email at a social provider and gain access to your primary account.

---

## 🏢 Enterprise Connections

Enterprise connections allow users from organizations to authenticate via their corporate identity provider (IdP).

### Supported Enterprise Connection Types

| Type | Strategy | Protocol | Use Case |
|------|----------|----------|----------|
| **Azure AD** | `waad` | OpenID Connect / SAML | Microsoft 365 organizations |
| **SAML** | `samlp` | SAML 2.0 | Okta, OneLogin, PingFederate, ADFS |
| **Active Directory / LDAP** | `ad` | LDAP via AD/LDAP Connector | On-premises AD |
| **Google Workspace** | `google-apps` | OpenID Connect | Google Workspace organizations |
| **OpenID Connect** | `oidc` | OIDC | Any OIDC-compliant IdP |
| **PingFederate** | `pingfederate` | SAML / OIDC | PingFederate servers |
| **Okta Workforce** | `ip` | OIDC | Okta Workforce Identity |
| **ADFS** | `adfs` | WS-Federation / SAML | Microsoft ADFS servers |

### Setting Up Azure AD

1. Go to **Dashboard > Authentication > Enterprise**
2. Click **"+ Create Connection"** next to **Microsoft Azure AD**
3. Enter:
   - **Connection name**: e.g., `acme-azure-ad`
   - **Microsoft Azure AD Domain**: `acme.onmicrosoft.com`
   - **Client ID**: From Azure AD app registration
   - **Client Secret**: From Azure AD app registration
4. Configure **permissions** (scopes requested from Azure AD)
5. Enable for your applications
6. **Test** the connection

#### Azure AD App Registration (on Microsoft side)

1. Go to [Azure Portal](https://portal.azure.com/) > **Azure Active Directory** > **App Registrations**
2. Click **"New registration"**
3. Enter:
   - Name: "Auth0 Integration"
   - Redirect URI: `https://YOUR_AUTH0_DOMAIN/login/callback`
4. Copy **Application (client) ID** and **Directory (tenant) ID**
5. Go to **Certificates & secrets** > **New client secret** → copy the value
6. Go to **API permissions** > Add: `openid`, `profile`, `email`, `User.Read`

### Setting Up a SAML Connection

SAML (Security Assertion Markup Language) is the most common enterprise SSO protocol.

1. Go to **Dashboard > Authentication > Enterprise**
2. Click **"+ Create Connection"** next to **SAML**
3. Enter:
   - **Connection name**: e.g., `acme-okta-saml`
   - **Sign In URL**: The IdP's SSO endpoint (e.g., `https://acme.okta.com/app/abc123/sso/saml`)
   - **Sign Out URL**: The IdP's SLO endpoint (optional)
   - **X.509 Signing Certificate**: Upload the IdP's certificate (PEM format)
4. Configure **attribute mappings** (map IdP attributes to Auth0 user profile fields)
5. Configure **protocol bindings**: HTTP-POST or HTTP-Redirect
6. Enable for applications and test

#### Configuring Okta as a SAML IdP

On the Okta side:
1. Log into **Okta Admin Console**
2. Go to **Applications > Create App Integration**
3. Select **SAML 2.0**
4. Configure:
   - Single Sign-On URL: `https://YOUR_AUTH0_DOMAIN/login/callback?connection=CONNECTION_NAME`
   - Audience URI (SP Entity ID): `urn:auth0:YOUR_TENANT:CONNECTION_NAME`
   - Attribute Statements: Map `email`, `firstName`, `lastName`
5. Download the **IdP Metadata** or copy the SSO URL and certificate

#### Configuring OneLogin as a SAML IdP

1. Log into **OneLogin**
2. Go to **Apps > Add App** > Search "SAML Test Connector (IdP w/attr)"
3. Configure:
   - ACS URL: `https://YOUR_AUTH0_DOMAIN/login/callback?connection=CONNECTION_NAME`
   - Audience: `urn:auth0:YOUR_TENANT:CONNECTION_NAME`
4. Copy **SAML 2.0 Endpoint** and **X.509 Certificate**

### Setting Up Active Directory / LDAP

For on-premises Active Directory, Auth0 uses the **AD/LDAP Connector** — a lightweight bridge that runs in your network.

1. Go to **Dashboard > Authentication > Enterprise > Active Directory / LDAP**
2. Click **"+ Create Connection"**
3. Download and install the **AD/LDAP Connector** on a server with access to your AD
4. Configure the connector:

```json
// config.json (AD/LDAP Connector)
{
  "AD_HUB": "https://YOUR_AUTH0_DOMAIN",
  "TENANT_SIGNING_KEY": "your-signing-key",
  "PROVISIONING_TICKET": "your-provisioning-ticket",
  "CONNECTION": "your-connection-name",
  "LDAP_URL": "ldap://your-dc.acme.com",
  "LDAP_BASE": "DC=acme,DC=com",
  "LDAP_BIND_USER": "CN=auth0-connector,OU=ServiceAccounts,DC=acme,DC=com",
  "LDAP_BIND_CREDENTIALS": "service-account-password"
}
```

5. The connector communicates **outbound** to Auth0 (no inbound firewall rules needed)
6. Test the connection by logging in with an AD account

### Setting Up Google Workspace

1. Go to **Dashboard > Authentication > Enterprise > Google Workspace**
2. Enter your **Google Workspace domain** (e.g., `acme.com`)
3. Enter **Client ID** and **Client Secret** from Google Cloud Console
4. Select attributes to sync (email, name, groups)
5. Enable for applications

---

## 📱 Passwordless Connections

Passwordless authentication eliminates passwords entirely. Users authenticate via a **magic link** or **one-time code** sent to their email or phone.

### Email Passwordless

1. Go to **Dashboard > Authentication > Passwordless**
2. Enable **Email**
3. Configure:
   - **Authentication method**: Magic Link or Code
   - **Email template**: Customize the email content
   - **Code length**: 4-8 digits
   - **Code expiration**: 5 minutes default

### SMS Passwordless

1. Enable **SMS**
2. Configure:
   - **Twilio** account (SID, Auth Token, Messaging Service SID)
   - **Message template**: Customize the SMS text
   - **Code length and expiration**

### Using Passwordless in Your App

```javascript
// Express — configure passwordless
const config = {
  authRequired: false,
  auth0Logout: true,
  secret: process.env.SECRET,
  baseURL: process.env.BASE_URL,
  clientID: process.env.CLIENT_ID,
  issuerBaseURL: process.env.ISSUER_BASE_URL,
  authorizationParams: {
    connection: 'email',  // Use email passwordless
    // OR
    // connection: 'sms', // Use SMS passwordless
  },
};
```

```tsx
// React — passwordless with auth0-react
const { loginWithRedirect } = useAuth0();

// Login with email magic link
loginWithRedirect({
  authorizationParams: {
    connection: 'email',
  },
});

// Login with SMS
loginWithRedirect({
  authorizationParams: {
    connection: 'sms',
  },
});
```

### Passwordless Flow

```
Email Magic Link:
  User enters email → Auth0 sends email with magic link →
  User clicks link → Auth0 verifies → User is logged in

Email Code:
  User enters email → Auth0 sends email with 6-digit code →
  User enters code → Auth0 verifies → User is logged in

SMS:
  User enters phone number → Auth0 sends SMS with 6-digit code →
  User enters code → Auth0 verifies → User is logged in
```

---

## 🏠 Home Realm Discovery

Home Realm Discovery (HRD) automatically routes users to the correct identity provider based on their **email domain**.

### How It Works

1. User enters their email on the Universal Login page
2. Auth0 extracts the domain (e.g., `@acme.com`)
3. Auth0 checks if any enterprise connection is configured for that domain
4. If found, the user is redirected to that IdP (e.g., Okta, Azure AD)
5. If not found, the user sees the default login options

### Configuring HRD

When you create an enterprise connection, you specify the **domain**:

1. Edit the enterprise connection (e.g., SAML or Azure AD)
2. Under **Login Experience** tab, enter the **email domain(s)**:
   - e.g., `acme.com`, `acme.co.uk`
3. Enable **Identity First Authentication** (shows email input first, then routes to IdP)

### HRD via the SDK

You can also trigger HRD programmatically:

```javascript
// Force a specific connection based on email domain
const { loginWithRedirect } = useAuth0();

const handleLogin = (email) => {
  const domain = email.split('@')[1];

  // Map domains to connections
  const domainConnectionMap = {
    'acme.com': 'acme-azure-ad',
    'bigcorp.com': 'bigcorp-okta-saml',
  };

  const connection = domainConnectionMap[domain];

  loginWithRedirect({
    authorizationParams: {
      connection: connection || undefined, // undefined = show all options
      login_hint: email,
    },
  });
};
```

### HRD with the `connection` Parameter

```
https://YOUR_AUTH0_DOMAIN/authorize
  ?response_type=code
  &client_id=YOUR_CLIENT_ID
  &redirect_uri=http://localhost:3000/callback
  &scope=openid profile email
  &connection=acme-azure-ad    ← Skip Universal Login, go directly to Azure AD
  &login_hint=user@acme.com    ← Pre-fill the email at the IdP
```

> [!tip] Identity-First Login
> With Identity-First Authentication enabled, the Universal Login page shows **only an email field** first. After the user enters their email, Auth0 determines the correct connection (HRD) and either shows the password field or redirects to the enterprise IdP.

---

## ⚙️ Connection-Level Settings and Mapping

### Attribute Mapping

For enterprise connections, you can map IdP attributes to Auth0 user profile fields:

#### SAML Attribute Mapping

```json
{
  "mappings": {
    "email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
    "given_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname",
    "family_name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname",
    "name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
    "nickname": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name",
    "groups": "http://schemas.xmlsoap.org/claims/Group"
  }
}
```

#### Azure AD Attribute Mapping

```json
{
  "email": "mail",
  "given_name": "givenName",
  "family_name": "surname",
  "name": "displayName",
  "picture": "photo",
  "department": "department",
  "job_title": "jobTitle"
}
```

### Connection-Level Settings

Each connection type has specific settings:

| Setting | Social | Enterprise | Database |
|---------|--------|------------|----------|
| **Enabled apps** | Yes | Yes | Yes |
| **Scopes/Permissions** | Yes | Some | N/A |
| **Domain mapping (HRD)** | No | Yes | No |
| **Attribute mapping** | Limited | Full | N/A |
| **Sync user profile** | Yes | Yes | N/A |
| **Auto-assign membership** | N/A | Yes (Orgs) | N/A |
| **Sign up disabled** | Via Rules/Actions | Via settings | Yes |
| **Password policy** | N/A | N/A | Yes |
| **Import users** | N/A | N/A | Yes |
| **Custom scripts** | N/A | N/A | Yes |
| **IP restrictions** | No | Some | No |

### Sync Frequency

- **Social**: User profile is synced on every login
- **Enterprise**: User profile can be synced on every login or on first login only
- **Database**: No sync needed (Auth0 is the source of truth)

### Multiple Connections for One App

An application can have **multiple connections** enabled:

- Database connection for email/password users
- Google social connection for consumer users
- Azure AD enterprise connection for corporate users
- Passwordless for low-friction login

Auth0's Universal Login page will show all enabled connections, with HRD routing enterprise users to their IdP automatically.

---

## 📊 Connection Type Decision Matrix

| Scenario | Recommended Connection |
|----------|----------------------|
| Consumer app (B2C) | Social (Google, Facebook) + Database |
| Internal tool | Enterprise (Azure AD, Okta) |
| B2B SaaS | Organizations + Enterprise per customer |
| Developer platform | Social (GitHub, Google) + Database |
| Healthcare / Regulated | Enterprise (SAML) + MFA |
| Mobile first | Social + Passwordless (SMS) |
| Government / High security | Enterprise (SAML) + WebAuthn MFA |
| Quick prototype | Auth0 Database + Development Keys |

---

## 🔗 Related Notes

- [[auth0-lesson-07-hands-on-setup]]
- [[auth0-lesson-08-web-app-integration]]
- [[auth0-lesson-09-rules-actions-hooks]]
- [[auth0-lesson-10-roles-permissions-rbac]]
- [[auth0-lesson-11-security-best-practices]]

## 📖 Sources

- [Auth0 Enterprise Connections](https://auth0.com/docs/authenticate/enterprise-connections)
- [Connect Your App to SAML Identity Providers](https://auth0.com/docs/authenticate/identity-providers/enterprise-identity-providers/saml)
- [Configure Okta as SAML IdP](https://dev.auth0.com/docs/authenticate/protocols/saml/saml-sso-integrations/configure-auth0-saml-service-provider/configure-okta-as-saml-identity-provider)
- [Google Social Connection Lab](https://developer.auth0.com/resources/labs/authentication/google-social-connection-to-login)
- [Google Sign-in and Authorization](https://auth0.com/ai/docs/google-sign-in-and-auth)
- [Auth0 Custom Google Social Connection](https://dev.to/victorioberra/auth0-custom-google-social-login-connection-1egg)
- [Integrating Auth0 with Okta SSO SAML](https://medium.com/@prem__kumar/integrating-auth0-with-okta-sso-saml-response-4514c2dac4a3)
- [Integrating Auth0 with OneLogin SSO SAML](https://medium.com/@prem__kumar/integrating-auth0-with-onelogin-sso-saml-response-28294e7abff5)
- [Azure AD Integration with Auth0](https://www.macnica.co.jp/en/business/security/manufacturers/okta/tech_auth0_azure_ad.html)
- [Auth0 How-To Videos — Azure AD, SAML, Custom Domains](https://auth0.com/blog/introducing-developer-success-how-to-videos/)
- [Multiple Organization Architecture](https://auth0.com/docs/get-started/architecture-scenarios/multiple-organization-architecture)
- [Invite Organization Members](https://dev.auth0.com/docs/manage-users/organizations/configure-organizations/invite-members)

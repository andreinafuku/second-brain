---
title: "Auth0 Lesson 7 — Hands-on: Setup"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - auth0
  - identity
  - status/rascunho
area: arquitetura
---

# 🔧 Auth0 Lesson 7 — Hands-on: Setup

## 🎯 Overview

This lesson walks through the practical setup of an Auth0 tenant, registering applications, configuring connections, and testing authentication with Universal Login. Everything here is foundational — you need this before integrating Auth0 into any application.

---

## 🏠 How to Create an Auth0 Tenant

A **tenant** is an isolated environment in Auth0. Each tenant has its own set of applications, connections, users, and rules. You typically create separate tenants for `dev`, `staging`, and `production`.

### Step-by-Step

1. **Sign up** at [auth0.com](https://auth0.com) (free tier available)
2. During signup, you choose your **tenant name** (e.g., `mycompany-dev`)
   - This becomes your domain: `mycompany-dev.us.auth0.com`
   - Tenant names are globally unique and **cannot be changed** after creation
3. Select a **region** (US, EU, AU, JP) — this determines where user data is stored
4. To create additional tenants:
   - Click the **tenant dropdown** (top-left corner of the Dashboard)
   - Select **"Create tenant"** at the bottom
   - Enter name, select region, choose environment tag (Development, Staging, Production)

### Tenant Best Practices

- Use **separate tenants** for each environment (dev/staging/prod)
- Name them consistently: `myapp-dev`, `myapp-staging`, `myapp-prod`
- Production tenants get **higher rate limits** and **no development-mode banners**
- Use **environment tags** to differentiate (Dashboard > Settings > General)

> [!tip] Tenant Regions
> Choose the region closest to your users. Data residency regulations (GDPR, LGPD) may require specific regions. Once set, the region **cannot be changed**.

---

## 📱 How to Register an Application

Applications in Auth0 represent the software that will use Auth0 for authentication. Navigate to **Dashboard > Applications > Applications > Create Application**.

### Application Types

| Type | Description | OAuth Flow | Example |
|------|-------------|------------|---------|
| **Single Page Application (SPA)** | JavaScript app running in the browser | Authorization Code + PKCE | React, Angular, Vue |
| **Regular Web Application** | Server-side rendered app | Authorization Code | Express.js, Django, Rails |
| **Native** | Mobile or desktop app | Authorization Code + PKCE | iOS, Android, Electron |
| **Machine-to-Machine (M2M)** | Backend service, daemon, CLI | Client Credentials | Cron jobs, microservices |

### Registering a SPA

1. Go to **Dashboard > Applications > Applications**
2. Click **"+ Create Application"**
3. Enter a name (e.g., "My React App")
4. Select **"Single Page Web Applications"**
5. Click **Create**
6. Go to the **Settings** tab
7. Note your **Domain**, **Client ID** (you will NOT get a Client Secret for SPAs — they are public clients)
8. Configure URLs (see section below)

### Registering a Regular Web Application

1. Same steps as above but select **"Regular Web Applications"**
2. You will get a **Client ID** AND a **Client Secret** (keep it secure!)
3. This type uses the standard Authorization Code flow (with a server-side exchange)

### Registering a Machine-to-Machine Application

1. Select **"Machine to Machine Applications"**
2. **Select the API** this application will access (e.g., Auth0 Management API)
3. **Select the scopes** (permissions) the application needs
4. You get a **Client ID** and **Client Secret** for the Client Credentials flow

> [!warning] Client Secrets
> SPAs and Native apps are **public clients** — they do NOT have a Client Secret. Only Regular Web Apps and M2M apps are **confidential clients** with secrets. Never expose a Client Secret in frontend code.

### Using the Auth0 CLI

```bash
# Install the CLI
npm install -g auth0-cli

# Login
auth0 login

# Create a Regular Web App
auth0 apps create \
  --name "My Express App" \
  --type regular \
  --callbacks http://localhost:3000/callback \
  --logout-urls http://localhost:3000 \
  --origins http://localhost:3000

# Create a SPA
auth0 apps create \
  --name "My React App" \
  --type spa \
  --callbacks http://localhost:3000 \
  --logout-urls http://localhost:3000 \
  --origins http://localhost:3000

# Create an M2M app
auth0 apps create \
  --name "My Backend Service" \
  --type m2m
```

---

## 🗄️ Configuring a Database Connection

A **Database Connection** stores user credentials (email/password) directly in Auth0. This is the default authentication method.

### Step-by-Step

1. Go to **Dashboard > Authentication > Database**
2. Click **"+ Create DB Connection"**
3. Enter a name (e.g., `Username-Password-Authentication`)
4. Configure settings:
   - **Requires Username**: Toggle if you want username + email (default: email only)
   - **Import Users**: Enable if migrating from another identity store (lazy migration)
   - **Disable Sign Ups**: Toggle to prevent self-registration
5. Go to the **Password Policy** tab:
   - Set password strength: `None`, `Low`, `Fair`, `Good`, `Excellent`
   - Configure custom rules (min length, special chars, etc.)
   - Enable **password history** (prevent reuse of last N passwords)
   - Enable **password dictionary** (block common passwords)
   - Enable **personal data check** (block passwords containing user info)
6. Go to the **Applications** tab:
   - Enable this connection for the applications that should use it
7. Go to the **Custom Database** tab (optional):
   - Write scripts to connect to an external database for authentication
   - Useful for **lazy migration** from legacy systems

### Password Policy Example

```
Strength: Good
Minimum Length: 10
Must contain: uppercase, lowercase, numbers, special characters
Password History: 5 (cannot reuse last 5 passwords)
Dictionary: Enabled
Personal Data: Enabled
```

> [!info] Lazy Migration
> With lazy migration, users are migrated one at a time as they log in. Auth0 calls your custom login script to validate credentials against your legacy database, then creates the user in Auth0. Subsequent logins hit Auth0 directly.

---

## 🌐 Configuring a Social Connection (Google Example)

Social connections let users authenticate with external identity providers like Google, GitHub, Facebook, etc.

### Setting Up Google Social Connection

#### Step 1: Create Google OAuth Credentials

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project (or select existing)
3. Navigate to **APIs & Services > Credentials**
4. Click **"+ Create Credentials" > OAuth Client ID**
5. If prompted, configure the **OAuth Consent Screen**:
   - User Type: **External**
   - App Name, Support Email, Developer Email
   - Add scopes: `openid`, `email`, `profile`
6. Create OAuth Client ID:
   - Application type: **Web application**
   - Authorized redirect URIs: `https://YOUR_AUTH0_DOMAIN/login/callback`
7. Copy the **Client ID** and **Client Secret**

#### Step 2: Configure in Auth0

1. Go to **Dashboard > Authentication > Social**
2. Click **"+ Create Connection"**
3. Select **Google / Gmail**
4. Enter your Google **Client ID** and **Client Secret**
5. Configure **Scopes** (permissions requested from Google):
   - `email` — user's email address
   - `profile` — basic profile info (name, picture)
   - Additional scopes as needed (e.g., `https://www.googleapis.com/auth/calendar.readonly`)
6. Under **Applications**, enable the connection for your apps
7. Click **Save**

#### Step 3: Test the Connection

1. On the connection page, click **"Try Connection"**
2. A new window opens with Google's login page
3. After authenticating, you should see the user's profile data returned

> [!tip] Development vs Production Google Credentials
> For development, Auth0 provides **built-in development keys** for Google (and other social providers). These have a "Powered by Auth0" consent screen. For production, you **must** create your own credentials in Google Cloud Console.

> [!warning] Google Verification
> If your app requests sensitive scopes or uses a custom logo, Google limits unverified apps to **100 logins**. Submit for verification early if you plan to go to production.

---

## 🔗 Setting Up Callback URLs and Allowed Origins

These settings are **critical** — misconfigured URLs are the #1 cause of authentication errors.

### Key URL Settings

Navigate to **Dashboard > Applications > [Your App] > Settings**:

| Setting | Purpose | Example |
|---------|---------|---------|
| **Allowed Callback URLs** | Where Auth0 redirects after login | `http://localhost:3000/callback` |
| **Allowed Logout URLs** | Where Auth0 redirects after logout | `http://localhost:3000` |
| **Allowed Web Origins** | Origins allowed for silent auth / token renewal (CORS) | `http://localhost:3000` |
| **Allowed Origins (CORS)** | Cross-origin requests to Auth0 APIs | `http://localhost:3000` |

### Configuration Rules

- **Multiple URLs**: Separate with commas (`,`)
- **Wildcards**: Use `*` for subdomains (e.g., `https://*.myapp.com/callback`) — but NOT recommended for production
- **No trailing slashes**: `http://localhost:3000` not `http://localhost:3000/`
- **Protocol matters**: `http://` and `https://` are different origins
- **Port matters**: `localhost:3000` and `localhost:5173` are different origins

### Example Configuration for Development

```
Allowed Callback URLs:
  http://localhost:3000/callback, http://localhost:5173

Allowed Logout URLs:
  http://localhost:3000, http://localhost:5173

Allowed Web Origins:
  http://localhost:3000, http://localhost:5173
```

### Example Configuration for Production

```
Allowed Callback URLs:
  https://myapp.com/callback, https://www.myapp.com/callback

Allowed Logout URLs:
  https://myapp.com, https://www.myapp.com

Allowed Web Origins:
  https://myapp.com, https://www.myapp.com
```

> [!danger] Common Errors
> - **"Callback URL mismatch"** — The redirect_uri in your app does not match any Allowed Callback URL. Check for typos, trailing slashes, port differences.
> - **"Invalid state"** — Often caused by mismatched callback URLs or CORS issues.
> - **Silent auth failures** — Missing Allowed Web Origins.

---

## 🧪 Testing with Auth0's Universal Login

**Universal Login** is Auth0's hosted login page. Instead of building your own login form, you redirect users to Auth0's login page, which handles all authentication logic.

### What is Universal Login?

- A **hosted login page** managed by Auth0
- Supports all connection types (database, social, enterprise)
- Handles MFA, password reset, signup
- Customizable via Dashboard or code
- Follows the **redirect-based** authentication model (recommended by OAuth 2.0 / OIDC)

### How to Test

#### Method 1: Try Connection

1. Go to **Dashboard > Authentication > Database** (or Social)
2. Click on your connection
3. Click **"Try Connection"**
4. This opens Universal Login in a new window
5. Complete the login flow
6. View the returned user profile and tokens

#### Method 2: Application Quick Start

1. Go to **Dashboard > Applications > [Your App]**
2. Click the **"Quick Start"** tab
3. Follow the interactive tutorial for your tech stack
4. The tutorial includes a **"Try"** button to test authentication

#### Method 3: Authentication API Explorer

1. Go to **Dashboard > Applications > APIs > Auth0 Management API**
2. Use the **API Explorer** tab to test token generation

#### Method 4: Direct URL Test

Construct the authorization URL manually:

```
https://YOUR_AUTH0_DOMAIN/authorize
  ?response_type=code
  &client_id=YOUR_CLIENT_ID
  &redirect_uri=http://localhost:3000/callback
  &scope=openid profile email
  &state=RANDOM_STATE_VALUE
```

### Customizing Universal Login

#### Basic Branding (Dashboard)

1. Go to **Dashboard > Branding > Universal Login**
2. Customize:
   - **Logo**: Upload your company logo
   - **Primary color**: Set button and link colors
   - **Background color**: Set page background
   - **Font**: Select from available fonts

#### Page Templates (Advanced)

Use the Auth0 CLI to customize the HTML template:

```bash
# Download current template
auth0 universal-login templates show

# Update template
auth0 universal-login templates update
```

Page templates use **Liquid templating** syntax and allow you to:

- Add custom headers/footers
- Inject analytics scripts
- Customize the entire page layout
- Add conditional rendering

#### Advanced Customizations (ACUL)

For full control, use ACUL (Advanced Customizations for Universal Login):

- Write code in your preferred framework (React, Vue, etc.)
- Full control over the UX
- Integrate A/B testing and analytics platforms

### Universal Login Experiences

| Feature | New Experience | Classic Experience |
|---------|---------------|-------------------|
| **Rendering** | Server-side rendered | Client-side SPA |
| **Customization** | Branding + Page Templates + ACUL | Full HTML/JS/CSS |
| **MFA support** | Built-in | Manual |
| **Passwordless** | Built-in | Manual |
| **WCAG compliance** | Yes (from July 2025) | Limited |
| **Recommended** | Yes | Legacy |

> [!tip] Always Use Universal Login
> Building a custom login form (embedded login) is **not recommended**. Universal Login is more secure (no credential exposure in your app), supports all features, and is maintained by Auth0. The redirect-based flow is the OAuth 2.0 / OIDC best practice.

---

## 📚 Key Concepts Summary

| Concept | Description |
|---------|-------------|
| **Tenant** | Isolated Auth0 environment (dev/staging/prod) |
| **Application** | Software that uses Auth0 for auth (SPA, Web, M2M) |
| **Connection** | How users authenticate (Database, Social, Enterprise) |
| **Universal Login** | Auth0's hosted login page (recommended) |
| **Callback URL** | Where Auth0 redirects after authentication |
| **Client ID** | Public identifier for your application |
| **Client Secret** | Confidential key for server-side apps only |

---

## 🔗 Related Notes

- [[auth0-lesson-08-web-app-integration]]
- [[auth0-lesson-09-rules-actions-hooks]]
- [[auth0-lesson-10-roles-permissions-rbac]]
- [[auth0-lesson-11-security-best-practices]]
- [[auth0-lesson-12-social-enterprise-connections]]

## 📖 Sources

- [Auth0 Documentation — Applications](https://auth0.com/docs/get-started/applications)
- [Auth0 CLI — Create Apps](https://auth0.github.io/auth0-cli/auth0_apps_create.html)
- [Auth0 Universal Login](https://auth0.com/docs/authenticate/login/auth0-universal-login)
- [Auth0 New Universal Login Experience](https://auth0.com/docs/authenticate/login/auth0-universal-login/new-experience)
- [Advanced Customizations for Universal Login](https://auth0.com/docs/customize/login-pages/advanced-customizations)
- [Create M2M Applications](https://auth0.com/docs/tokens/management-api-access-tokens/create-and-authorize-a-machine-to-machine-application)
- [Google Social Connection Lab](https://developer.auth0.com/resources/labs/authentication/google-social-connection-to-login)

---
title: "Auth0 Lesson 3 — Authentication Protocols"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - status/rascunho
area: authentication
---

# 🌐 Auth0 Lesson 3 — Authentication Protocols

## 🎯 Context

Understanding the three major authentication/authorization protocols that Auth0 supports: OAuth 2.0, OpenID Connect (OIDC), and SAML. Knowing when and why to use each is essential for making architectural decisions.

## 🔑 OAuth 2.0

### What is OAuth 2.0?

OAuth 2.0 is an **authorization framework** (not an authentication protocol) defined in [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749). It enables third-party applications to obtain limited access to a resource on behalf of a user, **without exposing the user's credentials**.

> [!important] Key Distinction
> OAuth 2.0 is about **authorization** ("what can this app access?"), not **authentication** ("who is this user?"). Authentication was bolted on later via OpenID Connect.

### The Four Roles in OAuth 2.0

```
┌─────────────────┐     ┌──────────────────────┐
│  Resource Owner  │     │  Authorization Server │
│  (The User)      │     │  (Auth0)              │
└────────┬────────┘     └──────────┬───────────┘
         │                         │
         │  Authorizes             │  Issues tokens
         │                         │
┌────────▼────────┐     ┌──────────▼───────────┐
│  Client          │     │  Resource Server      │
│  (Your App)      │     │  (Your API)           │
└─────────────────┘     └──────────────────────┘
```

| Role | Description | Example |
|------|-------------|---------|
| **Resource Owner** | The entity that owns the protected resource (usually the user) | A user who owns their profile data |
| **Client** | The application requesting access to the resource | Your web app, mobile app, or SPA |
| **Authorization Server** | The server that authenticates the resource owner and issues tokens | Auth0 (`your-tenant.auth0.com`) |
| **Resource Server** | The server hosting the protected resources (the API) | Your backend API (`api.myapp.com`) |

### Grant Types Overview

OAuth 2.0 defines several **grant types** (flows) for different scenarios:

| Grant Type | Use Case | Details |
|-----------|----------|---------|
| **Authorization Code** | Server-side apps | Most secure; exchanges a code for tokens |
| **Authorization Code + PKCE** | SPAs, mobile apps | Adds Proof Key for Code Exchange for public clients |
| **Client Credentials** | M2M / service-to-service | No user involved; app authenticates directly |
| **Device Authorization** | TVs, IoT, CLI tools | User authorizes on a separate device |
| **Implicit** (legacy) | Was used for SPAs | **Deprecated** — tokens exposed in URL fragment |
| **Resource Owner Password** (legacy) | Direct username/password | **Deprecated** — sends credentials directly to app |

> [!note] Grant Types are Covered in Detail
> See [[auth0-lesson-04-auth-flows]] for step-by-step walkthroughs of each flow.

### Key OAuth 2.0 Concepts

- **Scopes** — Permissions the client is requesting (e.g., `read:users`, `write:posts`)
- **Consent** — The user explicitly agrees to grant the requested scopes
- **Redirect URI** — Where the authorization server sends the user after authentication
- **State Parameter** — A random value to prevent CSRF attacks
- **Access Token** — The credential used to access the resource server
- **Refresh Token** — Used to obtain new access tokens without re-authentication

## 🆔 OpenID Connect (OIDC)

### What is OIDC?

OpenID Connect is an **identity layer built on top of OAuth 2.0**. While OAuth 2.0 handles authorization, OIDC adds standardized **authentication** — it answers "who is the user?"

```
┌──────────────────────────────────────────────────┐
│                  OpenID Connect                    │
│  (Authentication: "Who are you?")                  │
│                                                    │
│  ┌──────────────────────────────────────────────┐ │
│  │              OAuth 2.0                        │ │
│  │  (Authorization: "What can you access?")      │ │
│  └──────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

### How OIDC Extends OAuth 2.0

| Feature | OAuth 2.0 | OIDC |
|---------|-----------|------|
| **Purpose** | Authorization | Authentication + Authorization |
| **Token returned** | Access Token | Access Token + **ID Token** |
| **User identity** | Not standardized | Standardized via ID Token claims |
| **Discovery** | Not standardized | **Well-known endpoint** (`/.well-known/openid-configuration`) |
| **User info** | Not standardized | **UserInfo endpoint** (`/userinfo`) |
| **Scopes** | Application-defined | Standardized scopes (`openid`, `profile`, `email`) |

### ID Tokens

The ID Token is a **JWT** (JSON Web Token) that contains claims about the authenticated user:

```json
{
  "iss": "https://my-tenant.auth0.com/",
  "sub": "auth0|507f1f77bcf86cd799439011",
  "aud": "abc123clientid",
  "exp": 1681324800,
  "iat": 1681321200,
  "name": "Alice Smith",
  "email": "alice@example.com",
  "picture": "https://example.com/alice.jpg",
  "email_verified": true,
  "nonce": "random_nonce_value"
}
```

> [!warning] ID Token vs. Access Token
> The **ID Token** is for the **client application** — it tells the app who the user is. The **Access Token** is for the **API/Resource Server** — it tells the API what the app is allowed to do. Never send the ID Token to an API as a credential.

### UserInfo Endpoint

OIDC defines a standard endpoint to retrieve user profile information:

```bash
GET https://my-tenant.auth0.com/userinfo
Authorization: Bearer {ACCESS_TOKEN}
```

Response:
```json
{
  "sub": "auth0|507f1f77bcf86cd799439011",
  "name": "Alice Smith",
  "email": "alice@example.com",
  "email_verified": true,
  "picture": "https://example.com/alice.jpg"
}
```

### OIDC Scopes

Standard OIDC scopes determine what claims are included:

| Scope | Claims Included |
|-------|----------------|
| `openid` | **Required** — Returns `sub` (subject identifier) |
| `profile` | `name`, `family_name`, `given_name`, `middle_name`, `nickname`, `preferred_username`, `picture`, `website`, `gender`, `birthdate`, `zoneinfo`, `locale`, `updated_at` |
| `email` | `email`, `email_verified` |
| `address` | `address` (formatted, street, locality, region, postal code, country) |
| `phone` | `phone_number`, `phone_number_verified` |

```javascript
// Requesting OIDC scopes in Auth0
const auth0 = await createAuth0Client({
  domain: 'my-tenant.auth0.com',
  clientId: 'abc123',
  authorizationParams: {
    scope: 'openid profile email',  // Standard OIDC scopes
    audience: 'https://api.myapp.com'
  }
});
```

### OIDC Discovery

Auth0 publishes an OIDC discovery document at:

```
GET https://my-tenant.auth0.com/.well-known/openid-configuration
```

This document contains:
- `issuer` — The issuer identifier
- `authorization_endpoint` — URL for the authorization endpoint
- `token_endpoint` — URL for the token endpoint
- `userinfo_endpoint` — URL for the UserInfo endpoint
- `jwks_uri` — URL for the JSON Web Key Set (for verifying token signatures)
- `scopes_supported` — List of supported scopes
- `response_types_supported` — Supported response types
- `grant_types_supported` — Supported grant types

## 📄 SAML (Security Assertion Markup Language)

### What is SAML?

SAML (Security Assertion Markup Language) is an **XML-based standard** for exchanging authentication and authorization data between parties. It predates OAuth 2.0 and is widely used in **enterprise environments**.

SAML version 2.0 was released in 2005 and remains the dominant version in use.

### SAML Roles

| Role | Description | In Auth0 Context |
|------|-------------|-----------------|
| **Identity Provider (IdP)** | The system that authenticates users and provides identity assertions | Auth0 (when acting as IdP), or an enterprise IdP like Azure AD |
| **Service Provider (SP)** | The application that relies on the IdP for authentication | Your application, or Auth0 (when accepting SAML from an upstream IdP) |

> [!info] Auth0 Can Be Both
> Auth0 can act as a **SAML IdP** (providing SAML assertions to your app) or use an external **SAML IdP** as a connection (accepting SAML assertions from Azure AD, Okta, etc.).

### How SAML Works

```
┌──────────┐          ┌──────────────┐          ┌──────────────┐
│  User     │          │  Service      │          │  Identity     │
│  (Browser)│          │  Provider     │          │  Provider     │
│           │          │  (Your App)   │          │  (Auth0/IdP)  │
└─────┬─────┘          └──────┬───────┘          └──────┬───────┘
      │                       │                         │
      │ 1. Access app         │                         │
      │──────────────────────►│                         │
      │                       │                         │
      │ 2. Redirect with SAML AuthnRequest              │
      │◄──────────────────────│                         │
      │                       │                         │
      │ 3. Forward AuthnRequest to IdP                  │
      │────────────────────────────────────────────────►│
      │                       │                         │
      │ 4. IdP authenticates user (login page)          │
      │◄───────────────────────────────────────────────►│
      │                       │                         │
      │ 5. IdP sends SAML Response (assertion) to SP    │
      │◄────────────────────────────────────────────────│
      │                       │                         │
      │ 6. Browser POSTs SAML Response to SP ACS URL    │
      │──────────────────────►│                         │
      │                       │                         │
      │ 7. SP validates assertion, creates session       │
      │◄──────────────────────│                         │
      │                       │                         │
      │ 8. User is logged in  │                         │
      │◄──────────────────────│                         │
```

### SAML Assertions

A SAML Assertion is an XML document containing:

1. **Authentication Statement** — Confirms the user was authenticated, when, and how
2. **Attribute Statement** — Contains user attributes (name, email, roles)
3. **Authorization Decision Statement** — (optional) What the user is authorized to do

```xml
<!-- Simplified SAML Assertion -->
<saml:Assertion>
  <saml:Issuer>https://my-tenant.auth0.com/</saml:Issuer>
  <saml:Subject>
    <saml:NameID>alice@example.com</saml:NameID>
  </saml:Subject>
  <saml:Conditions NotBefore="2026-04-13T00:00:00Z"
                   NotOnOrAfter="2026-04-13T01:00:00Z">
    <saml:AudienceRestriction>
      <saml:Audience>https://myapp.com</saml:Audience>
    </saml:AudienceRestriction>
  </saml:Conditions>
  <saml:AuthnStatement AuthnInstant="2026-04-13T00:05:00Z">
    <saml:AuthnContext>
      <saml:AuthnContextClassRef>
        urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
      </saml:AuthnContextClassRef>
    </saml:AuthnContext>
  </saml:AuthnStatement>
  <saml:AttributeStatement>
    <saml:Attribute Name="email">
      <saml:AttributeValue>alice@example.com</saml:AttributeValue>
    </saml:Attribute>
    <saml:Attribute Name="name">
      <saml:AttributeValue>Alice Smith</saml:AttributeValue>
    </saml:Attribute>
  </saml:AttributeStatement>
</saml:Assertion>
```

### SAML Key Concepts

| Concept | Description |
|---------|-------------|
| **ACS URL** | Assertion Consumer Service URL — where the SP receives the SAML Response |
| **Entity ID** | Unique identifier for the IdP or SP |
| **Metadata XML** | XML document describing the IdP/SP configuration (endpoints, certificates) |
| **Signing Certificate** | X.509 certificate used to sign assertions (ensures integrity) |
| **Binding** | How SAML messages are transported (HTTP-POST, HTTP-Redirect) |
| **RelayState** | A value that the SP sends to the IdP and gets back unchanged (similar to OAuth `state`) |

## 🤔 When to Use Each Protocol

| Scenario | Protocol | Reason |
|----------|----------|--------|
| **Modern web/mobile app** | OIDC (OAuth 2.0) | JSON-based, lightweight, excellent SDK support |
| **API authorization** | OAuth 2.0 | Designed for delegated API access |
| **Enterprise SSO** | SAML | Most enterprise IdPs (Azure AD, Okta, ADFS) support it natively |
| **B2B SaaS with enterprise customers** | SAML + OIDC | SAML for enterprise connections, OIDC for your app |
| **Microservices** | OAuth 2.0 (Client Credentials) | M2M communication with scoped access tokens |
| **Legacy systems** | SAML | Many older enterprise systems only support SAML |
| **New green-field project** | OIDC | More modern, simpler, better developer experience |

### Protocol Comparison Summary

| Feature | OAuth 2.0 | OIDC | SAML |
|---------|-----------|------|------|
| **Format** | JSON | JSON (JWT) | XML |
| **Transport** | HTTPS | HTTPS | HTTP-POST / HTTP-Redirect |
| **Year** | 2012 | 2014 | 2005 |
| **Primary use** | Authorization | Authentication + Authorization | Authentication + SSO |
| **Token type** | Access Token (opaque or JWT) | ID Token (JWT) | SAML Assertion (XML) |
| **Best for** | API access | Modern web/mobile identity | Enterprise federation |
| **Complexity** | Medium | Medium (built on OAuth 2.0) | High (XML, certificates, metadata) |
| **Mobile-friendly** | Yes | Yes | Limited (XML parsing on mobile is painful) |

## 🔧 How Auth0 Supports All Three

### Auth0 as OIDC Provider

This is the **default and most common** usage. Your application redirects to Auth0's Universal Login, Auth0 authenticates the user, and returns OIDC tokens (ID Token + Access Token):

```
Your App → Auth0 (OIDC Provider) → Returns JWT tokens
```

### Auth0 as SAML IdP

Auth0 can serve SAML assertions to Service Providers. This is useful when:
- Your app needs to integrate with systems that only accept SAML
- You want Auth0 to be the central IdP for multiple SAML-consuming apps

```
SAML SP (e.g., Salesforce) → Auth0 (SAML IdP) → Authenticates user → Returns SAML Assertion
```

### Auth0 Consuming SAML from an External IdP

Auth0 can accept SAML assertions from an external enterprise IdP as a **connection**:

```
Your App → Auth0 → Redirects to Enterprise SAML IdP (e.g., Azure AD) → Auth0 receives SAML Assertion → Converts to OIDC tokens → Returns to your app
```

> [!tip] Protocol Translation
> This is one of Auth0's most powerful features: it can **accept SAML** from an enterprise IdP and **return OIDC tokens** to your app. Your app only needs to implement OIDC, regardless of how many enterprise customers use SAML internally.

### Auth0 as OAuth 2.0 Authorization Server

Auth0 issues OAuth 2.0 access tokens for API authorization:

```
Your App → Auth0 (Authorization Server) → Issues Access Token → Your API validates the token
```

## 🔗 Sources

- [What's the Difference Between OAuth, OpenID Connect, and SAML? — Okta](https://www.okta.com/identity-101/whats-the-difference-between-oauth-openid-connect-and-saml/)
- [SAML Configuration — Auth0 Docs](https://auth0.com/docs/authenticate/protocols/saml/saml-configuration)
- [OIDC vs SAML: When to Use Each for Modern SSO](https://www.authgear.com/post/oidc-vs-saml)
- [How OpenID Connect Works — OpenID Foundation](https://openid.net/developers/how-connect-works/)
- [RFC 6749 — The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)

## 📝 Related Notes

- [[auth0-lesson-01-what-is-auth0]]
- [[auth0-lesson-02-core-concepts]]
- [[auth0-lesson-04-auth-flows]]
- [[auth0-lesson-05-tokens]]
- [[auth0-lesson-06-jwt-deep-dive]]

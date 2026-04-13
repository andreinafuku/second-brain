---
title: "Auth0 Lesson 6 — JWT Deep Dive"
date: 2026-04-13
tags:
  - pesquisa
  - arquitetura/security
  - status/rascunho
area: authentication
---

# 🔬 Auth0 Lesson 6 — JWT Deep Dive

## 🎯 Context

JSON Web Tokens (JWTs) are the backbone of modern authentication. This lesson dissects JWTs completely: their anatomy, encoding, claims, signing algorithms, verification process, JWKS, and security pitfalls.

## 🧬 Anatomy of a JWT

A JWT consists of three parts separated by dots (`.`):

```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwczovL215LXRlbmFudC5hdXRoMC5jb20vIiwic3ViIjoiYXV0aDB8NTA3ZjFmNzdiY2Y4NmNkNzk5NDM5MDExIiwiYXVkIjoiaHR0cHM6Ly9hcGkubXlhcHAuY29tIiwiZXhwIjoxNjgxNDA3NjAwLCJpYXQiOjE2ODEzMjEyMDAsInNjb3BlIjoib3BlbmlkIHByb2ZpbGUgZW1haWwifQ.signature_bytes_here
```

```
┌─────────────┐.┌─────────────┐.┌─────────────┐
│   HEADER     │ │   PAYLOAD    │ │  SIGNATURE   │
│  (metadata)  │ │  (claims)    │ │  (integrity) │
└─────────────┘ └─────────────┘ └─────────────┘
```

### 1. Header

The header specifies the token type and signing algorithm:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "NjVBRjY5MDlCMUIwNzU4RTA2QzZFMDQ4QzQ2MDAyQjVDNjk1RTM2Qg"
}
```

| Field | Meaning |
|-------|---------|
| `alg` | Signing algorithm (RS256, HS256, etc.) |
| `typ` | Token type (always "JWT") |
| `kid` | Key ID — identifies which key was used to sign (for key rotation) |

### 2. Payload

The payload contains **claims** — statements about the user and metadata:

```json
{
  "iss": "https://my-tenant.auth0.com/",
  "sub": "auth0|507f1f77bcf86cd799439011",
  "aud": "https://api.myapp.com",
  "exp": 1681407600,
  "iat": 1681321200,
  "scope": "openid profile email",
  "https://myapp.com/claims/role": "admin"
}
```

### 3. Signature

The signature ensures the token hasn't been tampered with and was issued by a trusted party.

For **RS256**:
```
RSASHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  privateKey
)
```

For **HS256**:
```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  sharedSecret
)
```

## 🔤 Base64URL Encoding

JWTs use **Base64URL** encoding (not standard Base64):

| Standard Base64 | Base64URL |
|----------------|-----------|
| Uses `+` | Uses `-` |
| Uses `/` | Uses `_` |
| Uses `=` padding | No padding |

```javascript
// Decoding a JWT payload (Node.js)
const token = 'eyJhbGciOiJS...';
const [headerB64, payloadB64, signature] = token.split('.');

const header = JSON.parse(
  Buffer.from(headerB64, 'base64url').toString('utf-8')
);
const payload = JSON.parse(
  Buffer.from(payloadB64, 'base64url').toString('utf-8')
);

console.log(header);
// { alg: 'RS256', typ: 'JWT', kid: '...' }

console.log(payload);
// { iss: 'https://...', sub: 'auth0|...', exp: ..., ... }
```

> [!warning] Decoding is NOT Verifying
> Anyone can decode a JWT — it's just Base64. **Decoding does not prove authenticity.** You must always **verify the signature** before trusting the claims.

## 📋 Common Claims

### Registered Claims (RFC 7519)

| Claim | Name | Description | Example |
|-------|------|-------------|---------|
| `iss` | Issuer | Who issued the token | `"https://my-tenant.auth0.com/"` |
| `sub` | Subject | Who the token is about (user ID) | `"auth0\|507f1f77bcf86cd799439011"` |
| `aud` | Audience | Who the token is intended for | `"https://api.myapp.com"` or `["api1", "api2"]` |
| `exp` | Expiration Time | When the token expires (Unix timestamp) | `1681407600` |
| `iat` | Issued At | When the token was issued (Unix timestamp) | `1681321200` |
| `nbf` | Not Before | Token not valid before this time | `1681321200` |
| `jti` | JWT ID | Unique token identifier (for replay prevention) | `"a1b2c3d4-e5f6-..."` |

### Auth0-Specific Claims

| Claim | Description |
|-------|-------------|
| `azp` | Authorized party — the client ID of the application |
| `scope` | Space-separated list of granted scopes |
| `permissions` | Array of specific permissions (when RBAC is enabled) |
| `gty` | Grant type used (e.g., `authorization_code`, `client-credentials`) |
| `org_id` | Organization ID (when using Auth0 Organizations) |

### Custom Claims and Namespacing

Auth0 requires custom claims to be **namespaced** with a URL to avoid collisions with standard claims:

```javascript
// Auth0 Action: Post Login — Adding custom claims
exports.onExecutePostLogin = async (event, api) => {
  // CORRECT: Namespaced custom claims
  api.accessToken.setCustomClaim(
    'https://myapp.com/claims/role', 'admin'
  );
  api.accessToken.setCustomClaim(
    'https://myapp.com/claims/tenant_id', 'org_abc123'
  );

  // WRONG: Non-namespaced claims will be SILENTLY IGNORED
  // api.accessToken.setCustomClaim('role', 'admin');  // ← Ignored!
};
```

**Rules for custom claims:**
- Must use a URL namespace you control (e.g., `https://myapp.com/claims/`)
- Cannot override registered claims (`iss`, `sub`, `aud`, `exp`, etc.)
- Cannot override standard OIDC claims in access tokens
- Maximum total custom claims payload: **100 KB**
- Custom claims added via Actions are automatically included in tokens issued via refresh tokens

## 🔏 Signing Algorithms: RS256 vs. HS256

### HS256 (HMAC-SHA256) — Symmetric

```
┌─────────────────────────────────────────┐
│              SAME SECRET KEY             │
│                                         │
│  Auth Server ──────── Your API          │
│  (signs)              (verifies)        │
│                                         │
│  Both parties share the same secret     │
└─────────────────────────────────────────┘
```

- **One key** for both signing and verification
- The secret must be shared between Auth0 and your API
- **Anyone with the secret can forge tokens**
- Faster computation (~10-50x faster than RS256)
- Suitable when signer and verifier are in the **same trust boundary**

```javascript
// HS256 signing (conceptual — Auth0 does this for you)
const crypto = require('crypto');

const header = base64url({ alg: 'HS256', typ: 'JWT' });
const payload = base64url({ sub: '123', exp: 1681407600 });
const signature = crypto
  .createHmac('sha256', 'my-shared-secret')
  .update(`${header}.${payload}`)
  .digest('base64url');

const jwt = `${header}.${payload}.${signature}`;
```

### RS256 (RSA-SHA256) — Asymmetric

```
┌─────────────────────────────────────────┐
│                                         │
│  Auth Server         Your API           │
│  (PRIVATE key)       (PUBLIC key)       │
│  Signs tokens        Verifies tokens    │
│                                         │
│  Only Auth Server can create tokens     │
│  Anyone with public key can verify      │
└─────────────────────────────────────────┘
```

- **Two keys**: private key (signs) and public key (verifies)
- Auth0 holds the private key; your API uses the public key
- **Only Auth0 can forge tokens** (only it has the private key)
- Public key is freely distributed via JWKS endpoint
- Enables **multiple APIs** to verify tokens without sharing secrets
- Supports seamless **key rotation** via JWKS

```javascript
// RS256 verification (conceptual)
const crypto = require('crypto');

const [headerB64, payloadB64, signatureB64] = jwt.split('.');
const data = `${headerB64}.${payloadB64}`;
const signature = Buffer.from(signatureB64, 'base64url');

const isValid = crypto.createVerify('RSA-SHA256')
  .update(data)
  .verify(publicKey, signature);  // publicKey from JWKS
```

### Comparison Table

| Aspect | HS256 | RS256 |
|--------|-------|-------|
| **Cryptography** | Symmetric (shared secret) | Asymmetric (public/private key pair) |
| **Who can sign** | Anyone with the secret | Only the private key holder |
| **Who can verify** | Anyone with the secret | Anyone with the public key |
| **Key distribution** | Secret must be securely shared | Public key openly distributed |
| **Key rotation** | Must distribute new secret to all services simultaneously | Publish new public key via JWKS; old key remains during transition |
| **Performance** | Faster (~10-50x) | Slower (but <10ms for 2048-bit key) |
| **Security model** | Both parties equally trusted | Directional trust (only signer is authoritative) |
| **Best for** | Single app, same trust boundary | Distributed systems, multiple APIs, microservices |
| **Auth0 default** | No (legacy) | **Yes (recommended)** |

> [!tip] Use RS256
> Auth0 defaults to RS256 for new applications. Use RS256 unless you have a very specific reason to use HS256. RS256 is more secure for distributed systems and supports JWKS-based key rotation.

## ✅ How to Verify a JWT — Step by Step

### Complete Verification Process

```javascript
const jwt = require('jsonwebtoken');
const jwksClient = require('jwks-rsa');

// Step 1: Parse the JWT (DO NOT TRUST YET)
const [headerB64, payloadB64, signatureB64] = token.split('.');
const header = JSON.parse(Buffer.from(headerB64, 'base64url').toString());
const payload = JSON.parse(Buffer.from(payloadB64, 'base64url').toString());

// Step 2: Check the algorithm
// CRITICAL: Enforce expected algorithm, don't trust the header
if (header.alg !== 'RS256') {
  throw new Error('Unexpected algorithm');
}

// Step 3: Get the signing key from JWKS
const client = jwksClient({
  jwksUri: 'https://my-tenant.auth0.com/.well-known/jwks.json',
  cache: true,           // Cache keys for performance
  rateLimit: true,       // Prevent abuse
  cacheMaxAge: 600000    // 10 minutes
});

function getKey(header, callback) {
  client.getSigningKey(header.kid, (err, key) => {
    const signingKey = key.publicKey || key.rsaPublicKey;
    callback(null, signingKey);
  });
}

// Step 4: Verify signature + claims
jwt.verify(token, getKey, {
  algorithms: ['RS256'],           // Only accept RS256
  audience: 'https://api.myapp.com', // Expected audience
  issuer: 'https://my-tenant.auth0.com/', // Expected issuer
  clockTolerance: 5                // 5 seconds tolerance for clock skew
}, (err, decoded) => {
  if (err) {
    // Token is invalid
    console.error('Token verification failed:', err.message);
    return;
  }
  // Token is valid — decoded contains the payload
  console.log('Verified payload:', decoded);
});
```

### Verification Checklist

| Step | Check | What Happens If Skipped |
|------|-------|------------------------|
| **1. Decode** | Parse header to get `alg` and `kid` | Cannot determine how to verify |
| **2. Algorithm** | Enforce expected algorithm (RS256) | Algorithm confusion attack possible |
| **3. Get Key** | Fetch public key from JWKS using `kid` | Cannot verify signature |
| **4. Signature** | Verify the cryptographic signature | Tampered tokens accepted |
| **5. Expiration** | Check `exp` > current time | Expired tokens accepted |
| **6. Not Before** | Check `nbf` <= current time (if present) | Premature tokens accepted |
| **7. Issuer** | Check `iss` matches expected value | Tokens from other issuers accepted |
| **8. Audience** | Check `aud` contains your API identifier | Tokens meant for other APIs accepted |
| **9. Scopes** | Check `scope` or `permissions` for required access | Unauthorized actions allowed |

> [!danger] Never Skip Steps
> Each verification step prevents a specific class of attacks. Skipping any step creates a security vulnerability.

## 🔑 JWKS (JSON Web Key Set)

### What is JWKS?

JWKS (JSON Web Key Set) is a standard format for publishing the **public keys** used to verify JWT signatures. Auth0 publishes its JWKS at:

```
https://my-tenant.auth0.com/.well-known/jwks.json
```

### JWKS Structure

```json
{
  "keys": [
    {
      "kty": "RSA",
      "use": "sig",
      "n": "0vx7agoebGcQSuuPiLJXZptN9nndrQmbXEps2aiAFb...",
      "e": "AQAB",
      "kid": "NjVBRjY5MDlCMUIwNzU4RTA2QzZFMDQ4QzQ2MDAyQjVDNjk1RTM2Qg",
      "x5t": "NjVBRjY5MDlCMUIwNzU4RTA2QzZFMDQ4QzQ2MDAyQjVDNjk1RTM2Qg",
      "x5c": [
        "MIIDBTCCA..."
      ],
      "alg": "RS256"
    },
    {
      "kty": "RSA",
      "use": "sig",
      "n": "another_key_modulus...",
      "e": "AQAB",
      "kid": "previous-key-id",
      "alg": "RS256"
    }
  ]
}
```

| Field | Meaning |
|-------|---------|
| `kty` | Key type (RSA, EC, etc.) |
| `use` | Key usage (`sig` = signature) |
| `n` | RSA modulus (the public key component) |
| `e` | RSA exponent (usually `AQAB` = 65537) |
| `kid` | Key ID — matches the `kid` in the JWT header |
| `x5t` | X.509 certificate thumbprint |
| `x5c` | X.509 certificate chain |
| `alg` | Algorithm this key is used with |

### Key Rotation

Auth0 supports key rotation — periodically changing the signing keys:

```
Timeline:
Day 0:  Auth0 signs with Key-A (kid: "key-a")
        JWKS contains: [Key-A]

Day 30: Auth0 rotates keys
        Auth0 signs with Key-B (kid: "key-b")
        JWKS contains: [Key-B, Key-A]  ← Both keys available!

Day 60: Auth0 removes Key-A
        JWKS contains: [Key-B]
```

**Why both keys exist during transition:**
- Tokens signed with Key-A are still valid (not expired)
- APIs fetching JWKS can verify both old and new tokens
- The `kid` in the JWT header tells the verifier which key to use

### JWKS Best Practices

```javascript
// GOOD: Cache JWKS with automatic refresh
const client = jwksClient({
  jwksUri: 'https://my-tenant.auth0.com/.well-known/jwks.json',
  cache: true,              // Cache keys in memory
  cacheMaxEntries: 5,       // Max keys to cache
  cacheMaxAge: 600000,      // 10 min cache TTL
  rateLimit: true,          // Rate limit JWKS fetches
  jwksRequestsPerMinute: 10 // Max 10 requests per minute
});

// BAD: Fetching JWKS on every request (performance killer)
// BAD: Caching indefinitely (won't pick up rotated keys)
// BAD: Not using kid to select the correct key
```

## ⚠️ Common JWT Security Pitfalls

### 1. `alg: "none"` Attack

**The Attack**: The attacker sets the algorithm to `"none"` in the header, strips the signature, and the server accepts the token without verification.

```json
// Malicious JWT header
{ "alg": "none", "typ": "JWT" }
```

The resulting token: `eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiIxMjMifQ.`
(Note the trailing dot with no signature)

**Prevention**: Always enforce a specific algorithm; never accept `"none"`:

```javascript
// GOOD: Enforce algorithm
jwt.verify(token, key, { algorithms: ['RS256'] });

// BAD: Accept whatever the token says
jwt.verify(token, key); // Vulnerable!
```

### 2. Algorithm Confusion (Key Confusion) Attack

**The Attack**: The server expects RS256 but the attacker:
1. Obtains Auth0's **public key** (freely available from JWKS)
2. Creates a token with `"alg": "HS256"`
3. Signs it using the **public key as the HMAC secret**
4. If the server blindly trusts the `alg` header and uses the public key for HS256 verification, the signature matches!

```
Expected:  RS256 → Verify with PUBLIC key
Attacker:  HS256 → Sign with PUBLIC key (used as HMAC secret)
Server:    Reads alg=HS256 → Uses PUBLIC key as HMAC secret → MATCHES!
```

**Prevention**: Never trust the `alg` claim in the header. Always enforce the expected algorithm:

```javascript
// GOOD: Explicitly specify allowed algorithms
jwt.verify(token, getKey, {
  algorithms: ['RS256']  // ONLY accept RS256
});

// BAD: Let the library decide based on the header
jwt.verify(token, key);  // Trusts alg from header → vulnerable!
```

### 3. JWK Header Injection

**The Attack**: The attacker embeds a malicious public key directly in the JWT's `jwk` header parameter:

```json
{
  "alg": "RS256",
  "typ": "JWT",
  "jwk": {
    "kty": "RSA",
    "n": "attacker_controlled_key...",
    "e": "AQAB"
  }
}
```

If the server uses the `jwk` from the header to verify, the attacker controls both the key and the signature.

**Prevention**: Never use the `jwk` parameter from the token header for verification. Always fetch keys from a trusted JWKS endpoint.

### 4. JKU (JWK Set URL) Header Injection

**The Attack**: The attacker sets the `jku` header to point to their own JWKS endpoint:

```json
{
  "alg": "RS256",
  "jku": "https://attacker.com/.well-known/jwks.json"
}
```

**Prevention**: Never follow `jku` URLs from the token. Always use a hardcoded, trusted JWKS URL.

### 5. KID (Key ID) Injection

**The Attack**: The `kid` field is used in database lookups or file system paths:

```json
// Directory traversal
{ "kid": "../../../../../../dev/null" }

// SQL injection
{ "kid": "key1' UNION SELECT 'known_secret' -- " }
```

If `kid` is used unsafely, attackers can:
- Point to `/dev/null` (empty file) and use an empty-string HMAC secret
- Inject SQL to retrieve a known value as the secret

**Prevention**: Validate `kid` against a whitelist. Never use it directly in file paths or SQL queries.

### 6. Expired Token Acceptance

**The Attack**: Accepting tokens that have passed their `exp` time.

**Prevention**: Always check `exp`:

```javascript
// Most libraries do this automatically, but verify:
const now = Math.floor(Date.now() / 1000);
if (payload.exp < now) {
  throw new Error('Token expired');
}
```

### 7. Audience Confusion

**The Attack**: A valid token for API-A is used to access API-B.

**Prevention**: Always verify `aud` matches your specific API:

```javascript
jwt.verify(token, key, {
  audience: 'https://api.myapp.com'  // Reject tokens for other APIs
});
```

### 8. Missing Signature Verification

**The Attack**: Developer uses `jwt.decode()` instead of `jwt.verify()`, skipping signature verification entirely.

```javascript
// CATASTROPHIC BUG
const payload = jwt.decode(token);  // Just decodes, NO verification!
// Attacker can put anything in the payload

// CORRECT
const payload = jwt.verify(token, key, { algorithms: ['RS256'] });
```

### Security Pitfalls Summary

| Pitfall | Prevention |
|---------|-----------|
| `alg: "none"` | Enforce specific algorithm in verification options |
| Algorithm confusion | Never trust the `alg` header; whitelist algorithms |
| JWK header injection | Never use embedded `jwk` from token headers |
| JKU injection | Hardcode JWKS URL; never follow `jku` from tokens |
| KID injection | Whitelist `kid` values; sanitize for path traversal/SQL injection |
| Expired tokens | Always check `exp` claim |
| Audience confusion | Always verify `aud` matches your API |
| Missing verification | Use `verify()` not `decode()`; never skip signature check |
| Weak HS256 secrets | Use minimum 256-bit (32-byte) random secret |
| Key leakage | Rotate keys immediately if compromised |

## 🛠️ Practical: Debugging JWTs

### Using jwt.io

[jwt.io](https://jwt.io) is a web tool for decoding and verifying JWTs:

1. Paste the JWT
2. View decoded header and payload
3. Paste the public key or secret to verify the signature

> [!warning] Security
> Do not paste production tokens with sensitive data into jwt.io. Use it only for development/testing tokens.

### Using the Command Line

```bash
# Decode a JWT header
echo 'eyJhbGciOiJSUzI1NiJ9' | base64 -d 2>/dev/null
# {"alg":"RS256"}

# Decode a JWT payload
echo 'eyJpc3MiOiJodHRwczovLy4uLiJ9' | base64 -d 2>/dev/null
# {"iss":"https://..."}

# Fetch Auth0 JWKS
curl -s https://my-tenant.auth0.com/.well-known/jwks.json | jq '.'
```

### Using Node.js

```javascript
const jose = require('jose');

async function inspectToken(token) {
  // Decode without verification (for debugging only!)
  const { protectedHeader, payload } = jose.decodeJwt(token);
  console.log('Header:', protectedHeader);
  console.log('Payload:', payload);
  console.log('Expires:', new Date(payload.exp * 1000).toISOString());
  console.log('Issued:', new Date(payload.iat * 1000).toISOString());

  // Verify with JWKS
  const JWKS = jose.createRemoteJWKSet(
    new URL('https://my-tenant.auth0.com/.well-known/jwks.json')
  );

  try {
    const { payload: verified } = await jose.jwtVerify(token, JWKS, {
      issuer: 'https://my-tenant.auth0.com/',
      audience: 'https://api.myapp.com'
    });
    console.log('Token is VALID:', verified);
  } catch (err) {
    console.error('Token is INVALID:', err.message);
  }
}
```

## 🔗 Sources

- [RS256 vs HS256: A Deep Dive into JWT Signing Algorithms — WorkOS](https://workos.com/blog/rs256-vs-hs256-jwt-signing-algorithms)
- [JWT Attacks — PortSwigger Web Security Academy](https://portswigger.net/web-security/jwt)
- [Algorithm Confusion Attacks — PortSwigger](https://portswigger.net/web-security/jwt/algorithm-confusion)
- [Navigating RS256 and JWKS — Auth0 Blog](https://auth0.com/blog/navigating-rs256-and-jwks/)
- [JSON Web Token Claims — Auth0 Docs](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-token-claims)
- [RFC 7519 — JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)
- [JWT Tokens Explained: Structure, Security & Best Practices (2026)](https://bytepane.com/blog/jwt-tokens-explained/)
- [JSON Web Token Attacks and Vulnerabilities — Acunetix](https://www.acunetix.com/blog/articles/json-web-token-jwt-attacks-vulnerabilities/)
- [JWT: Vulnerabilities, Attacks & Security Best Practices — Vaadata](https://www.vaadata.com/blog/jwt-json-web-token-vulnerabilities-common-attacks-and-security-best-practices/)

## 📝 Related Notes

- [[auth0-lesson-01-what-is-auth0]]
- [[auth0-lesson-02-core-concepts]]
- [[auth0-lesson-03-authentication-protocols]]
- [[auth0-lesson-04-auth-flows]]
- [[auth0-lesson-05-tokens]]

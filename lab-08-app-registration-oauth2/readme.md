# Lab 08 — App Registration + OAuth2/OIDC

## Objective

Register an application in Microsoft Entra ID, configure delegated API permissions with admin consent, create a client secret as application credential, and explore the OAuth2/OIDC authentication endpoints. Understand the difference between delegated and application permissions, and how the authorization code flow works behind any "Sign in with Microsoft" button.

**Zero Trust principle applied:** Verify explicitly — every application that accesses resources must be explicitly registered, granted only the permissions it needs, and authenticated with a verifiable credential. No implicit trust is granted to unregistered apps.

---

## Environment

| Component | Value |
|---|---|
| Tenant | DidacIAMLab.onmicrosoft.com |
| Tenant ID | 1d043a45-4372-4f19-8368-a1fa9eeff496 |
| License | Microsoft Entra ID P2 (trial) |
| Admin account | DidacSanchez@DidacIAMLab.onmicrosoft.com |
| Portal | entra.microsoft.com |

---

## What Was Built

| Component | Name/Value | Purpose |
|---|---|---|
| App registration | IAM-Lab-WebApp | Represents a web application registered in Entra ID |
| Client ID | 67e03afd-fda0-4eb9-abb1-ed9288f71d35 | Unique identifier of the app in the tenant |
| Redirect URI | https://localhost | Where Entra ID sends the auth code after login |
| Delegated permissions | User.Read, User.ReadBasic.All, Group.Read.All | Scopes the app can request on behalf of a signed-in user |
| Client secret | IAM-Lab-Secret (expires 2/9/2027) | App credential used to authenticate against the token endpoint |

---

## Steps

### 1. Register the application

Navigated to **App registrations → + New registration** and configured:

| Setting | Value |
|---|---|
| Name | IAM-Lab-WebApp |
| Supported account types | Single tenant — Didac IAM Lab only |
| Redirect URI | Web → https://localhost |

Single tenant means only users from `DidacIAMLab.onmicrosoft.com` can authenticate. The redirect URI is where Entra ID sends the authorization code after a successful login — in production this would be a real HTTPS endpoint.

![App registration basics](01-app-registration-basics.png)

---

### 2. Review app identifiers

After registration, the Overview page shows the three critical identifiers used in every OAuth2/OIDC flow:

| Identifier | Value | Used for |
|---|---|---|
| Application (client) ID | 67e03afd-fda0-4eb9-abb1-ed9288f71d35 | Identifies the app to Entra ID |
| Directory (tenant) ID | 1d043a45-4372-4f19-8368-a1fa9eeff496 | Identifies which tenant to authenticate against |
| Object ID | e87396b2-a54b-4ed1-8ab7-adf08771fbf1 | Internal Entra ID object reference |

![App overview IDs](02-app-overview-ids.png)

---

### 3. Configure delegated API permissions

Navigated to **API permissions → + Add a permission → Microsoft Graph → Delegated permissions** and added:

| Permission | Type | Admin consent required | Description |
|---|---|---|---|
| User.Read | Delegated | No | Sign in and read the signed-in user's profile |
| User.ReadBasic.All | Delegated | No | Read basic profile of all users in the directory |
| Group.Read.All | Delegated | Yes | Read all groups in the directory |

**Delegated vs Application permissions:**
- **Delegated**: the app acts on behalf of a signed-in user — the user's own permissions act as a ceiling. If the user can't read a group, the app can't either.
- **Application**: the app acts as itself, with no user context — used for background services and daemons. More powerful and requires stricter governance.

![API permissions configured](03-api-permissions.png)

---

### 4. Grant admin consent

`Group.Read.All` requires admin consent because it allows reading all groups in the directory — a sensitive operation that individual users cannot consent to on their own.

Clicked **Grant admin consent for Didac IAM Lab** → confirmed. All three permissions now show a green tick with "Granted for Didac IA...".

Without admin consent, any user signing in through this app would see a consent prompt for Group.Read.All — and if user consent is blocked by policy (as it should be in production), the login would fail entirely.

![Admin consent granted](04-admin-consent-granted.png)

---

### 5. Create client secret

Navigated to **Certificates & secrets → Client secrets → + New client secret**:

| Setting | Value |
|---|---|
| Description | IAM-Lab-Secret |
| Expires | 180 days — 2/9/2027 |

The secret value is only shown once immediately after creation. After navigating away, only the Secret ID remains visible — the value is permanently hidden. In production, this value would be stored in Azure Key Vault, never in code or config files.

**Certificate vs Secret:**
Certificates are the recommended credential type for production — they use asymmetric cryptography and the private key never leaves the app server. Secrets are simpler but symmetric — if leaked, they grant full app access until rotated.

![Client secret created](05-client-secret.png)

---

### 6. OAuth2/OIDC endpoints

Clicked **Endpoints** in the Overview toolbar. The panel shows all authentication endpoints available for this tenant:

| Endpoint | URL pattern | Purpose |
|---|---|---|
| OAuth 2.0 authorization (v2) | .../oauth2/v2.0/authorize | Step 1: user logs in and consents, gets auth code |
| OAuth 2.0 token (v2) | .../oauth2/v2.0/token | Step 2: app exchanges auth code for access token |
| OpenID Connect metadata | .../.well-known/openid-configuration | Discovery document — lists all endpoints and signing keys |
| SAML-P sign-on | .../saml2 | Legacy SAML federation endpoint |
| Microsoft Graph API | https://graph.microsoft.com | Resource endpoint the access token is sent to |

All endpoints are tenant-scoped — they include the Tenant ID `1d043a45-4372-4f19-8368-a1fa9eeff496` in the URL, ensuring only users from this tenant can authenticate.

![OAuth2 endpoints](06-oauth2-endpoints.png)

---

## OAuth2 Authorization Code Flow — How It Works

This is the flow behind every "Sign in with Microsoft" button:

```
1. User clicks "Sign in with Microsoft" in the app
2. App redirects user to:
   https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize
   ?client_id=67e03afd-...
   &response_type=code
   &redirect_uri=https://localhost
   &scope=User.Read Group.Read.All

3. User authenticates (MFA if required by CA policy)
4. Entra ID sends authorization code to redirect_uri

5. App exchanges code for tokens at:
   POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
   Body: code=..., client_id=..., client_secret=..., redirect_uri=...

6. Entra ID returns:
   - access_token (used to call Graph API)
   - id_token (JWT with user identity claims)
   - refresh_token (used to get new access tokens silently)

7. App calls Microsoft Graph with the access token:
   GET https://graph.microsoft.com/v1.0/me
   Authorization: Bearer {access_token}
```

---

## Design Decisions

**Why single tenant instead of multi-tenant?**
Single tenant restricts authentication to `DidacIAMLab.onmicrosoft.com` only. Multi-tenant would allow users from any Microsoft Entra directory to sign in — appropriate for SaaS products but introduces additional security considerations (guest user policies, cross-tenant access settings). For a lab environment, single tenant is correct.

**Why use a client secret instead of a certificate?**
For lab purposes, secrets are simpler to create and don't require certificate infrastructure. In production, certificates are always preferred — they use asymmetric cryptography, the private key never leaves the server, and they can't be accidentally logged or leaked in plain text the way secrets can.

**Why grant admin consent at the tenant level instead of per-user?**
Per-user consent for `Group.Read.All` would fail in most production tenants because admin consent requirements block it by policy. Tenant-wide admin consent means all users get a seamless login experience — the consent dialog never appears. This is the standard pattern for internal corporate apps.

**Why https://localhost as the redirect URI?**
For a lab with no real web server, localhost is a safe placeholder. In production, the redirect URI must be an HTTPS endpoint owned by the app — Entra ID validates it against the registered list before sending the auth code, which prevents redirect URI hijacking attacks.

---

## Lessons Learned

**The client secret value is shown exactly once.** After navigating away from the Certificates & secrets page, the Value column shows only a partial string. There is no way to retrieve the full value — if lost, a new secret must be created and the old one deleted. In production this is why secrets go directly into Key Vault at creation time.

**Admin consent is separate from permission configuration.** Adding a permission to the app registration does not grant it — it only declares what the app *may* request. The actual grant happens through admin consent (for admin-required permissions) or user consent (for user-consentable permissions). An app with unconsented permissions will receive an error at runtime.

**v2 endpoints vs v1 endpoints.** Both appear in the endpoints panel. The v2 endpoint supports both work/school accounts and personal Microsoft accounts, uses scopes instead of resource URIs, and supports incremental consent. Always use v2 for new applications — v1 (ADAL-based) is in maintenance mode and Microsoft has deprecated ADAL.

---

## Key Concepts (SC-300 Relevant)

| Concept | Description |
|---|---|
| **App registration** | Represents an application's identity in Entra ID — the config object that defines what the app can do |
| **Service principal** | The runtime instance of an app registration in a specific tenant; created automatically on registration |
| **Client ID** | The app's unique identifier, included in every auth request so Entra ID knows which app is asking |
| **Redirect URI** | The endpoint where Entra ID sends the auth code; must match exactly what's registered to prevent hijacking |
| **Delegated permission** | Access granted on behalf of a signed-in user; bounded by the user's own permissions |
| **Application permission** | Access granted to the app itself with no user context; requires admin consent; used for daemons/services |
| **Admin consent** | Tenant-wide permission grant by an admin; eliminates per-user consent prompts for sensitive scopes |
| **Client secret** | A password-like credential the app uses to prove its identity when exchanging auth codes for tokens |
| **Access token** | Short-lived JWT (usually 1 hour) that the app presents to APIs as proof of authorization |
| **Authorization code flow** | The standard OAuth2 flow for web apps: user logs in → gets code → app exchanges code for tokens |
| **OpenID Connect (OIDC)** | OAuth2 extension that adds identity (id_token with user claims) on top of authorization |

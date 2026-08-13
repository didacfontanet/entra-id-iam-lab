# Lab 09 — Microsoft Graph API (Graph Explorer)

## Objective

Query Microsoft Entra ID programmatically using the Microsoft Graph API via Graph Explorer. Demonstrate how identity data (users, groups, memberships) can be retrieved and filtered through API calls rather than portal clicks — the foundation of IAM automation. Understand permission scopes, consent flows, and OData query parameters.

**Zero Trust principle applied:** Verify explicitly — every API call must be authenticated and authorized with explicit permission scopes. No data is accessible without a valid token and consented permissions, regardless of the caller's identity.

---

## Environment

| Component | Value |
|---|---|
| Tenant | DidacIAMLab.onmicrosoft.com |
| License | Microsoft Entra ID P2 (trial) |
| Admin account | DidacSanchez@DidacIAMLab.onmicrosoft.com |
| Tool | Microsoft Graph Explorer — developer.microsoft.com/graph/graph-explorer |
| API version | v1.0 |

> **Note:** Graph Explorer was used instead of PowerShell + Microsoft.Graph module due to restrictions on the work laptop. Graph Explorer provides identical API access through the browser — same endpoints, same tokens, same responses. In production environments, these queries would be scripted in PowerShell or Python for automation.

---

## Queries Executed

| # | Method | Endpoint | Result |
|---|---|---|---|
| 1 | GET | /v1.0/me | Admin profile JSON |
| 2 | GET | /v1.0/users | All 3 tenant users |
| 3 | GET | /v1.0/groups | All 7 tenant groups |
| 4 | GET | /v1.0/users?$select=...&$orderby=... | Filtered and sorted user list |
| 5 | GET | /v1.0/groups/{id}/members | Members of SG-Finance-App-Access |

---

## Steps

### 1. Get signed-in user profile — GET /me

First query: retrieve the profile of the currently authenticated user.

```http
GET https://graph.microsoft.com/v1.0/me
```

Response confirmed the admin identity with all standard user properties: `displayName`, `userPrincipalName`, `mail`, `id`, `preferredLanguage`.

![Graph GET me](01-graph-get-me.png)

---

### 2. List all users — GET /users

```http
GET https://graph.microsoft.com/v1.0/users
```

**Troubleshooting — 403 Forbidden on first attempt:**
Initial call returned `Authorization_RequestDenied` — Graph Explorer did not have the `User.ReadBasic.All` permission consented for this session.

Resolution: navigated to **Modify permissions** tab → found `User.ReadBasic.All` → clicked **Consent** → accepted the permission prompt. Re-ran the query successfully.

This error demonstrates a key concept: having admin rights in the tenant does not automatically grant Graph API permissions in a client application. Each app (including Graph Explorer) must explicitly consent to the scopes it needs.

![Graph 403 insufficient privileges](02b-graph-403-insufficient-privileges.png)

After consent, the query returned all 3 tenant users:

| User | UPN | Object ID |
|---|---|---|
| Break Glass Emergency | breakglass@DidacIAMLab.onmicrosoft.com | 2c76d6c6-17c9-46dc-b1a1-a79834286bd8 |
| Didac Sanchez | DidacSanchez@DidacIAMLab.onmicrosoft.com | 910a7e82-1773-4264-b6fa-f6bc3e903d29 |
| Test PIM User | testpim@DidacIAMLab.onmicrosoft.com | 6ce2cbb8-b316-4662-b728-e2b1c9d9e0ed |

![Graph GET users](02-graph-get-users.png)

---

### 3. List all groups — GET /groups

```http
GET https://graph.microsoft.com/v1.0/groups
```

**Troubleshooting — 403 Forbidden on first attempt:**
Same pattern as /users — `Group.Read.All` required a separate consent in Graph Explorer even though it had been granted tenant-wide on the `IAM-Lab-WebApp` registration in Lab 08. Graph Explorer is a different application with its own permission grants.

Resolution: **Modify permissions** → `Group.Read.All` → **Consent**.

![Graph groups 403](03b-graph-groups-403.png)

After consent, the query returned all 7 tenant groups:

| Group | Type | Purpose |
|---|---|---|
| SG-Privileged-App-Admins | Security (role-assignable) | PIM for groups — Lab 07 |
| Didac IAM Lab | Microsoft 365 | Default tenant group |
| All Company | Microsoft 365 | Default Viva Engage group |
| SG-AllUsers | Security | Standard users subject to CA policies |
| Group for Answers in Viva Engage | Microsoft 365 | Viva Engage system group |
| SG-CA-Excluded | Security | Break-glass exclusion from CA policies |
| SG-Finance-App-Access | Security | Access Package resource — Lab 06 |

![Graph GET groups](03-graph-get-groups.png)

---

### 4. Filtered query with OData parameters

```http
GET https://graph.microsoft.com/v1.0/users?$select=displayName,userPrincipalName,id&$orderby=displayName
```

Used OData query parameters to return only the fields needed, sorted alphabetically:
- `$select` — limits the properties returned (reduces payload, improves performance)
- `$orderby` — sorts results by displayName ascending

This is the pattern used in production scripts — never return full objects when you only need specific fields.

![Graph select orderby](04-graph-select-orderby.png)

---

### 5. Get group members by Object ID

```http
GET https://graph.microsoft.com/v1.0/groups/f61f9cd9-b880-4159-a781-89d0d212d9be/members
```

Used the Object ID of `SG-Finance-App-Access` (retrieved from the /groups response in step 3) to query its current members programmatically.

Response confirmed **Test PIM User** (`testpim@DidacIAMLab.onmicrosoft.com`) as a member — the same membership that was auto-provisioned by the Access Package approval flow in Lab 06, now visible via API.

This demonstrates the full circle: access was requested via myaccess portal → approved → provisioned automatically → now verifiable programmatically via Graph API.

![Graph group members](05-graph-group-members.png)

---

## Equivalent PowerShell Commands

These are the PowerShell equivalents of the Graph Explorer queries above. They would be run with the `Microsoft.Graph` module connected via `Connect-MgGraph -Scopes "User.Read.All","Group.Read.All"`:

```powershell
# 1. Get signed-in user profile
Get-MgContext

# 2. List all users
Get-MgUser -All | Select-Object DisplayName, UserPrincipalName, Id

# 3. List all groups
Get-MgGroup -All | Select-Object DisplayName, SecurityEnabled, Id

# 4. Filtered query with specific properties, sorted
Get-MgUser -All -Property DisplayName, UserPrincipalName, Id |
    Select-Object DisplayName, UserPrincipalName, Id |
    Sort-Object DisplayName

# 5. Get members of a specific group
Get-MgGroupMember -GroupId "f61f9cd9-b880-4159-a781-89d0d212d9be" |
    ForEach-Object {
        Get-MgUser -UserId $_.Id | Select-Object DisplayName, UserPrincipalName
    }
```

---

## Design Decisions

**Why Graph Explorer instead of PowerShell?**
The work laptop had restrictions preventing `Install-Module` from PSGallery. Graph Explorer provides identical REST API access through the browser — same Microsoft identity platform, same tokens, same endpoints. The underlying HTTP calls are what matter conceptually, not the client tool used to make them. In a production automation context, these would be scripted in PowerShell or Python.

**Why use v1.0 instead of beta?**
The v1.0 endpoint contains only stable, GA features with Microsoft's backwards compatibility guarantee. The beta endpoint has more features but breaking changes can happen without notice. For production scripts and lab documentation, v1.0 is always the correct choice unless a specific feature is only available in beta.

**Why query group members by Object ID instead of displayName?**
Object IDs are immutable — they never change even if the group is renamed. DisplayNames can be edited. Any script or automation that references a group by name will break silently if the group is renamed. Always use Object IDs in scripts and API calls.

**Why are the 403 errors documented instead of hidden?**
The permission consent flow is one of the most misunderstood aspects of Graph API in real environments. Many junior admins assume that being a Global Admin grants automatic access to all API calls — it does not. Each application must be explicitly granted the scopes it needs through consent. Documenting this error demonstrates understanding of the OAuth2 consent model, not a mistake.

---

## Lessons Learned

**Tenant-wide admin consent on an app registration does not carry over to other apps.** In Lab 08, `Group.Read.All` was granted on `IAM-Lab-WebApp`. Graph Explorer is a separate app registration — it required its own consent grant. This is correct behavior: each application has its own permission grants, isolated from other apps.

**OData query parameters drastically reduce payload size.** The full `/users` response includes ~20 properties per user. Using `$select=displayName,userPrincipalName,id` reduces this to 3 — for tenants with thousands of users, this is the difference between a fast script and one that times out or hits throttling limits.

**Object IDs from one query can be chained into the next.** The `/groups` response returned the Object ID of `SG-Finance-App-Access`, which was immediately used in the `/groups/{id}/members` query. This chaining pattern is fundamental to Graph API scripting — retrieve IDs first, then use them in subsequent calls.

**Labs connect across each other.** The group member returned in step 5 was provisioned by the Access Package in Lab 06. Seeing the same data through a different lens (portal vs API) reinforces understanding of what Entra ID is actually doing under the hood.

---

## Key Concepts (SC-300 Relevant)

| Concept | Description |
|---|---|
| **Microsoft Graph API** | Unified REST API for accessing all Microsoft 365 and Entra ID data — users, groups, devices, mail, calendar, and more |
| **Graph Explorer** | Browser-based tool for testing Graph API queries with your own tenant data |
| **Permission scope** | A named permission that grants access to a specific resource (e.g. `User.Read.All`, `Group.Read.All`) |
| **Delegated permission** | API access on behalf of a signed-in user — bounded by the user's own permissions |
| **Consent** | The act of a user or admin explicitly granting an app access to specific scopes |
| **OData $select** | Returns only specified properties — reduces payload and improves performance |
| **OData $filter** | Filters results server-side — e.g. `$filter=displayName eq 'Test PIM User'` |
| **OData $orderby** | Sorts results — e.g. `$orderby=displayName asc` |
| **Object ID** | Immutable GUID that uniquely identifies any Entra ID object — always use this in scripts, never displayName |
| **Bearer token** | The access token included in every Graph API request as `Authorization: Bearer {token}` |
| **v1.0 vs beta** | v1.0 = stable GA features with backwards compatibility; beta = preview features, may break without notice |

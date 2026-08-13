# Lab 06 — Access Packages (Entitlement Management)

## Objective

Configure an end-to-end entitlement management workflow using Microsoft Entra ID Access Packages. Demonstrate how users can self-request access to resources through a governed catalog, with mandatory approval, automatic provisioning, and time-bound expiration — eliminating manual IT intervention.

**Zero Trust principle applied:** Verify explicitly + Least privilege access — users receive only the access they request, only for the time they need it, through a documented and auditable approval chain.

---

## Environment

| Component | Value |
|---|---|
| Tenant | DidacIAMLab.onmicrosoft.com |
| License | Microsoft Entra ID P2 (trial) |
| Admin account | DidacSanchez@DidacIAMLab.onmicrosoft.com |
| Test user | testpim@DidacIAMLab.onmicrosoft.com |
| Portal | entra.microsoft.com / myaccess.microsoft.com |

---

## What Was Built

| Component | Name | Purpose |
|---|---|---|
| Resource group | SG-Finance-App-Access | Security group representing a finance application resource |
| Catalog | IAM-Lab-Catalog | Container for access packages and their resources |
| Access Package | AP-Finance-Access | Requestable bundle of access to finance resources |
| Policy | Initial Policy | Defines who can request, approval flow, and expiration |

---

## Steps

### 1. Create resource group

Created security group `SG-Finance-App-Access` (Assigned membership, no initial members) to serve as the resource that will be governed by the access package.

![Resource group created](01-resource-group-created.png)

---

### 2. Create catalog

Created `IAM-Lab-Catalog` under Identity Governance → Entitlement management → Catalogs.

- Enabled for users to request: **Yes**
- Enabled for external users: **No** (internal tenant only)

![Catalog created](02-catalog-created.png)

---

### 3. Add resource to catalog

Added `SG-Finance-App-Access` as a resource inside `IAM-Lab-Catalog` via the Resources tab.

![Catalog resources](03-catalog-resources.png)

---

### 4. Create access package — basics

Created `AP-Finance-Access` inside `IAM-Lab-Catalog`.

- Name: `AP-Finance-Access`
- Description: `Access to finance application resources`
- Catalog: `IAM-Lab-Catalog`

![Access package basics](04-access-package-basics.png)

---

### 5. Configure resource roles

Linked `SG-Finance-App-Access` to the access package with role **Member** — meaning approved users get added to the group automatically.

![Resource roles](05-resource-roles.png)

---

### 6. Configure request policy

Defined who can request access and the approval chain:

| Setting | Value |
|---|---|
| Who can request | SG-AllUsers (specific group in directory) |
| Who can request (self) | Self (enabled) |
| Require approval | Yes |
| Approval stages | 1 |
| First approver | DidacSanchez |
| Require requestor justification | Yes |
| Require approver justification | Yes |
| Decision deadline | 14 days |

![Request policy approval](06-request-policy-approval.png)

---

### 7. Configure lifecycle

Set automatic expiration to enforce time-bound access:

| Setting | Value |
|---|---|
| Assignments expire | Number of days |
| Days until expiration | 30 |
| Users can request specific timeline | No |
| Require access reviews | No |
| Allow users to extend access | Yes |
| Require approval to grant extension | Yes |

![Lifecycle expiration](07-lifecycle-expiration.png)

---

### 8. Access package created

`AP-Finance-Access` created successfully inside `IAM-Lab-Catalog`.

![Access package created](08-access-package-created.png)

---

### 9. End-user request via myaccess.microsoft.com

Logged in as `testpim` in a private browser session. Navigated to **myaccess.microsoft.com → Access packages → AP-Finance-Access → Request**.

Submitted business justification: *"Need access to finance resources for Q3 reporting"*

Request entered pending approval state immediately.

![MyAccess request from testpim](09-myaccess-request-testpim.png)

---

### 10. Admin approval

Logged in as `DidacSanchez` → **myaccess.microsoft.com → Approvals → Pending**.

Reviewed request from Test PIM User, selected **Approve**, and provided justification: *"Approved for Q3 finance project access"*

![Approval granted](10-approval-granted.png)

---

### 11. Automatic group provisioning verified

Navigated to **Groups → SG-Finance-App-Access → Members**.

Test PIM User appeared as a direct member within seconds of approval — no manual IT action required.

![Auto-provisioned membership](11-auto-provisioned-membership.png)

---

### 12. Assignment with expiration date confirmed

Navigated to **AP-Finance-Access → Assignments**.

Assignment visible with:
- Status: **Delivered**
- End date: **September 12, 2026** (30 days from assignment)
- Access will be automatically revoked on that date

![Assignment with expiry](12-assignment-with-expiry.png)

---

## Design Decisions

**Why a dedicated catalog instead of using the built-in General catalog?**
Catalogs are the governance boundary in Entitlement Management — separate catalogs allow different ownership, delegation, and policy per department or application domain. Using a dedicated catalog demonstrates understanding of multi-team governance, not just the default path.

**Why SG-AllUsers as requestable scope instead of All members?**
Scoping to a specific group rather than all directory members follows least privilege at the policy level — only users who belong to SG-AllUsers can see and request this package. In production this would be scoped to the relevant department or team.

**Why 30-day expiration instead of permanent access?**
Time-bound access is a core Zero Trust control. Permanent access accumulates over time and creates orphaned permissions. A 30-day window forces periodic revalidation — users who still need access must request an extension (also requiring approval), creating an automatic audit trail.

**Why require both requestor and approver justification?**
Dual justification creates an auditable record on both sides. In an audit or security review, you can trace not just who approved access but why it was requested and why it was granted — this is the difference between a rubber-stamp approval process and governed access.

---

## Lessons Learned

**Self checkbox must be explicitly enabled.** In the Requests policy, the "Who can request access" section has a separate "Self" checkbox that is not selected by default. Without enabling it, end users cannot request access themselves — only admins can assign directly. This is a common misconfiguration in production environments.

**Propagation delay is expected in trial tenants.** Group membership after approval appeared almost immediately in this lab (~30 seconds), but the portal documentation warns of up to 15 minutes in production tenants. This is normal Azure AD replication behavior and not an error.

**The My Access portal link is tenant-specific.** The direct link (`https://myaccess.microsoft.com/@DidacIAMLab.onmicrosoft.com`) scopes the portal to the specific tenant, which is useful in multi-tenant environments where a user might belong to multiple directories.

---

## Key Concepts (SC-300 Relevant)

| Concept | Description |
|---|---|
| **Entitlement Management** | Azure AD feature that automates access request, approval, and lifecycle for groups, apps, and SharePoint sites |
| **Access Package** | A named bundle of resource roles that users can request as a unit |
| **Catalog** | A governance container that groups access packages and their resources; supports delegated management |
| **Policy** | Defines requestable scope, approval workflow, and lifecycle for an access package |
| **Auto-provisioning** | Automatic group membership assignment upon access package approval, without IT intervention |
| **Time-bound access** | Access that expires automatically after a defined period, enforcing least privilege over time |
| **myaccess.microsoft.com** | The self-service portal where end users request, view, and manage their access packages |
| **Separation of duties** | Access packages support SoD rules to prevent conflicting access combinations |

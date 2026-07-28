# Lab 05 — Access Reviews & entitlement management

## Objective

Configure periodic access reviews to verify that users still need their group memberships and role assignments. Set up an access package with entitlement management for governed access requests. This is the identity governance layer — the topic that separates IAM Engineers from IT Support.

## Environment

| | |
|---|---|
| **Tenant** | DidacIAMLab.onmicrosoft.com |
| **License required** | Entra ID P2 (Identity Governance) |
| **Users involved** | DidacSanchez (admin/reviewer), test.user01-03 |
| **Time** | ~2 hours |

## What I configured

### Step 1 — Create an access review for a security group

<!-- TODO: Add screenshot -->
<!-- ![Access review creation](./screenshots/01-access-review-creation.png) -->

Entra ID → Identity Governance → Access Reviews → New:

| Setting | Value |
|---------|-------|
| Review name | Quarterly review — SG-AllUsers |
| Scope | Members of SG-AllUsers |
| Reviewers | Group owners (self-review) |
| Frequency | Quarterly |
| Duration | 14 days |
| Auto-apply results | Yes |
| If reviewer doesn't respond | Remove access |

### Step 2 — Complete a review cycle

<!-- TODO: Add screenshot -->
<!-- ![Review in progress](./screenshots/02-review-in-progress.png) -->

1. Reviewer received email notification
2. Opened review → saw list of group members
3. For each member: Approve (still needs access) or Deny (remove)
4. Submitted decisions with justification

### Step 3 — Configure entitlement management — access package

<!-- TODO: Add screenshot -->
<!-- ![Access package](./screenshots/03-access-package.png) -->

Entra ID → Identity Governance → Entitlement management → Access packages → New:

| Setting | Value |
|---------|-------|
| Package name | IAM Lab Resources |
| Catalog | General |
| Resources | SG-AllUsers group membership |
| Who can request | All users in directory |
| Approval required | Yes (DidacSanchez) |
| Access expires | 90 days |

### Step 4 — Request access as a user

<!-- TODO: Add screenshot -->
<!-- ![Access request flow](./screenshots/04-access-request.png) -->

1. Signed in as `test.user01`
2. Went to myaccess.microsoft.com
3. Found "IAM Lab Resources" package → Request access
4. Entered justification
5. Request submitted → pending approval

### Step 5 — Approve and verify

<!-- TODO: Add screenshot -->
<!-- ![Approval and verification](./screenshots/05-approval.png) -->

1. Approved request as DidacSanchez
2. test.user01 automatically added to SG-AllUsers
3. Access will expire in 90 days (lifecycle managed)
4. Audit log shows complete request-approve-provision chain

## Lessons learned

1. **Access reviews answer the audit question "who has access and why?"** — without them, group memberships grow forever and nobody cleans them up
2. **"If reviewer doesn't respond → remove access" is the secure default** — silence should mean revocation, not approval
3. **Entitlement management is like an internal "app store" for access** — users request what they need, approvers verify, the system provisions and de-provisions automatically
4. **This is governance, not operations** — understanding this layer is what makes the difference between an IT Support role and an IAM Engineer role

## Key concepts for SC-300

- Access reviews require Entra ID P2
- Review scope: groups, applications, Entra roles, Azure resource roles
- Entitlement management: catalogs → access packages → policies
- Lifecycle management: expiry, auto-renewal, access removal
- Connected organizations for B2B access governance

---

*Lab completed: [DATE]*
*Tenant: DidacIAMLab.onmicrosoft.com*

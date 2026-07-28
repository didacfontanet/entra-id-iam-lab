# Lab 02 — Privileged Identity Management (PIM)

## Objective

Configure just-in-time privileged access using PIM: assign eligible roles instead of permanent ones, require justification and approval for activation, and verify the complete audit trail. PIM is one of the highest-weighted SC-300 topics and a key differentiator for IAM roles.

## Environment

| | |
|---|---|
| **Tenant** | DidacIAMLab.onmicrosoft.com |
| **License required** | Entra ID P2 |
| **Users involved** | DidacSanchez (admin/approver), testpim (eligible user) |
| **Time** | ~2 hours |

## What I configured

### Step 1 — Enable PIM and assign an eligible role

<!-- TODO: Add screenshot -->
<!-- ![PIM eligible role assignment](./screenshots/01-eligible-role.png) -->

Assigned **User Administrator** as an eligible role to `testpim@DidacIAMLab.onmicrosoft.com`.

**Eligible vs Active:**
- **Active** = the user HAS the role permanently (traditional, risky)
- **Eligible** = the user CAN activate the role when needed (just-in-time, secure)

This is the core of PIM: nobody has standing privileges. You request access when you need it.

### Step 2 — Configure role settings

<!-- TODO: Add screenshot -->
<!-- ![PIM role settings](./screenshots/02-role-settings.png) -->

| Setting | Value |
|---------|-------|
| Activation maximum duration | 4 hours |
| Require justification | Yes |
| Require approval | Yes |
| Approver | DidacSanchez (Global Admin) |
| Require MFA on activation | Yes |
| Notification on activation | Email to admins |

**Design decisions:**
- 4-hour window is enough for administrative tasks without leaving permanent access
- Requiring both justification and approval creates a paper trail for auditors
- MFA on activation adds a second verification layer at the moment of privilege escalation

### Step 3 — Simulate activation as testpim

<!-- TODO: Add screenshot -->
<!-- ![PIM activation request](./screenshots/03-activation-request.png) -->

1. Signed in as `testpim`
2. Navigated to Entra ID → Roles and administrators → My roles
3. Found "User Administrator" under Eligible roles
4. Clicked "Activate" → entered justification: "Need to reset user password for support ticket #1234"
5. Completed MFA challenge
6. Request submitted → waiting for approval

### Step 4 — Approve the activation

<!-- TODO: Add screenshot -->
<!-- ![PIM approval](./screenshots/04-approval.png) -->

1. Signed in as `DidacSanchez` (approver)
2. Received email notification of pending activation
3. Navigated to PIM → Approve requests
4. Reviewed justification and approved
5. `testpim` now has User Administrator for 4 hours

### Step 5 — Verify in audit logs

<!-- TODO: Add screenshot -->
<!-- ![PIM audit log](./screenshots/05-audit-log.png) -->

Confirmed in Entra ID → Audit logs:
- Role activation requested by testpim
- Justification recorded
- Approval by DidacSanchez
- Role activated with 4-hour expiry
- Full audit trail preserved

## Lessons learned

1. **PIM solves the "too many permanent admins" problem** — in SEIDOR I see tenants where 10+ users have permanent Global Admin. PIM eliminates that risk
2. **The justification field is critical for compliance** — auditors need to see WHY someone activated a privileged role
3. **4-hour activation is a good default** — long enough for real work, short enough to limit exposure
4. **This is the #1 feature that separates IAM roles from support roles** — knowing PIM signals you understand governance, not just helpdesk

## Key concepts for SC-300

- PIM requires Entra ID P2
- Eligible vs Active assignments
- Just-in-time (JIT) vs standing access
- Activation settings: duration, justification, approval, MFA
- PIM for Azure resources (separate from Entra roles)

---

*Lab completed: [DATE]*
*Tenant: DidacIAMLab.onmicrosoft.com*

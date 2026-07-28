# Lab 04 — SSPR & authentication methods

## Objective

Configure Self-Service Password Reset (SSPR) with multiple authentication methods and test the complete user experience. SSPR reduces helpdesk ticket volume by 20-40% in enterprise environments — it's one of the most practical IAM features to understand.

## Environment

| | |
|---|---|
| **Tenant** | DidacIAMLab.onmicrosoft.com |
| **License required** | Entra ID P1 (for group-based SSPR) |
| **Users involved** | DidacSanchez (admin), test.user01 (end user) |
| **Time** | ~1 hour |

## What I configured

### Step 1 — Enable SSPR

<!-- TODO: Add screenshot -->
<!-- ![SSPR configuration](./screenshots/01-sspr-config.png) -->

Entra ID → Password reset → Properties:
| Setting | Value |
|---------|-------|
| SSPR enabled | Selected group (SG-AllUsers) |
| Number of methods required | 2 |
| Methods available | Mobile phone, Email, Authenticator app |

### Step 2 — Configure authentication methods

<!-- TODO: Add screenshot -->
<!-- ![Authentication methods](./screenshots/02-auth-methods.png) -->

Entra ID → Security → Authentication methods:
- Microsoft Authenticator: Enabled for all users
- SMS: Enabled for all users
- Email OTP: Enabled for all users

### Step 3 — User registration experience

<!-- TODO: Add screenshot -->
<!-- ![SSPR registration](./screenshots/03-user-registration.png) -->

1. Signed in as `test.user01`
2. Redirected to security info registration (aka.ms/mysecurityinfo)
3. Registered phone number + email as SSPR methods
4. Registration complete — user can now reset their own password

### Step 4 — Test password reset flow

<!-- TODO: Add screenshot -->
<!-- ![Password reset flow](./screenshots/04-password-reset.png) -->

1. Went to login.microsoftonline.com → "Can't access your account?"
2. Entered test.user01 username
3. Verified identity using registered phone (SMS code)
4. Verified with second method (email)
5. Set new password → success

### Step 5 — Admin view: SSPR usage report

<!-- TODO: Add screenshot -->
<!-- ![SSPR usage report](./screenshots/05-sspr-report.png) -->

Entra ID → Password reset → Usage:
- Registration activity
- Reset activity
- Methods used breakdown

## Lessons learned

1. **SSPR directly reduces helpdesk workload** — at SEIDOR, password resets are one of the most common tickets. SSPR eliminates most of them
2. **Two methods required is the sweet spot** — one method is insecure (phone theft = account takeover), three is too burdensome for users
3. **Registration enforcement matters** — without it, users never set up SSPR and still call helpdesk
4. **Combined SSPR + MFA registration** saves users from registering twice — one flow covers both

## Key concepts for SC-300

- SSPR requires P1 for group-scoped, P2 not needed
- On-premises writeback needs Entra Connect (hybrid environments)
- SSPR + MFA combined registration is the modern default
- Authentication methods policy (centralized) vs per-feature settings (legacy)

---

*Lab completed: [DATE]*
*Tenant: DidacIAMLab.onmicrosoft.com*

# Lab 03 — Identity Protection & risky sign-ins

## Objective

Configure Identity Protection policies to detect and respond to risky sign-ins and compromised users. Simulate a risky sign-in using a VPN/Tor and verify that the system detects and remediates it. This lab produces the most visual screenshots — perfect for demonstrating threat detection skills.

## Environment

| | |
|---|---|
| **Tenant** | DidacIAMLab.onmicrosoft.com |
| **License required** | Entra ID P2 |
| **Users involved** | DidacSanchez (admin), test.user01 (target) |
| **Tools** | Tor Browser or VPN to simulate risky sign-in |
| **Time** | ~2 hours |

## What I configured

### Step 1 — Enable Identity Protection policies

<!-- TODO: Add screenshot -->
<!-- ![Identity Protection dashboard](./screenshots/01-identity-protection-dashboard.png) -->

Configured two policies:

**User risk policy:**
| Setting | Value |
|---------|-------|
| Users | All users |
| User risk level | Medium and above |
| Action | Require password change |

**Sign-in risk policy:**
| Setting | Value |
|---------|-------|
| Users | All users |
| Sign-in risk level | Medium and above |
| Action | Require MFA |

### Step 2 — Simulate a risky sign-in

<!-- TODO: Add screenshot -->
<!-- ![Risky sign-in simulation](./screenshots/02-risky-signin.png) -->

1. Downloaded Tor Browser (or connected to a foreign VPN)
2. Attempted to sign in as `test.user01` from the anonymous/foreign IP
3. Identity Protection flagged the sign-in as risky due to:
   - Unfamiliar location
   - Anonymous IP address (Tor) / atypical travel

### Step 3 — Review risk detections

<!-- TODO: Add screenshot -->
<!-- ![Risk detections](./screenshots/03-risk-detections.png) -->

Navigated to Entra ID → Security → Identity Protection → Risk detections:
- Detection type: Anonymous IP address / Unfamiliar sign-in properties
- Risk level: Medium
- User: test.user01
- Action taken: MFA required (per sign-in risk policy)

### Step 4 — Remediation flow

<!-- TODO: Add screenshot -->
<!-- ![Remediation](./screenshots/04-remediation.png) -->

- User was prompted to complete MFA to prove identity
- After successful MFA, sign-in risk was dismissed
- For user risk: password change was required before next sign-in

### Step 5 — Review in reports

<!-- TODO: Add screenshot -->
<!-- ![Risk reports](./screenshots/05-risk-reports.png) -->

Identity Protection reports show:
- Risky users count
- Risky sign-ins count
- Risk detections timeline
- Remediation status

## Lessons learned

1. **Identity Protection is the "security camera" of Entra ID** — it doesn't prevent attacks, it detects them and triggers automatic response
2. **Tor Browser is the easiest way to trigger a risk detection** in a lab — the anonymous IP is immediately flagged
3. **The combination of CA + Identity Protection is powerful** — CA can use risk level as a condition (e.g., "if sign-in risk is high → block access entirely")
4. **This is exactly what SOC teams monitor** — understanding risk detections makes you valuable for both IAM and security operations roles

## Key concepts for SC-300

- Identity Protection requires Entra ID P2
- User risk vs sign-in risk (different signals, different remediations)
- Risk levels: low, medium, high
- Integration with Conditional Access (risk-based CA policies)
- Risk detections: leaked credentials, anonymous IP, atypical travel, malware-linked IP

---

*Lab completed: [DATE]*
*Tenant: DidacIAMLab.onmicrosoft.com*

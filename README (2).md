# Microsoft Entra ID — Identity & Access Management Labs

Hands-on IAM labs built in a dedicated Microsoft Entra ID tenant, documenting real configurations, design decisions, troubleshooting, and audit verification for every identity scenario. Each lab maps to Zero Trust principles and SC-300 exam objectives.

---

## About

| | |
|---|---|
| **Author** | Didac Sanchez Fontanet |
| **Tenant** | DidacIAMLab.onmicrosoft.com |
| **License used** | Entra ID P2 (trial, Jul–Aug 2026) + Microsoft 365 Business Basic |
| **Stack** | Microsoft Entra ID · Conditional Access · PIM · Identity Protection · Identity Governance · Graph API · OAuth2/OIDC |
| **Certification roadmap** | SC-900 · SC-300 · AZ-500 |
| **Portfolio** | [didac-sf-portfolio.netlify.app](https://didac-sf-portfolio.netlify.app) |
| **LinkedIn** | [linkedin.com/in/didac-sf](https://linkedin.com/in/didac-sf) |

---

## Labs

| # | Lab | Status | Key topics |
|---|---|---|---|
| 01 | [Conditional Access — MFA policy from scratch](lab-01-conditional-access/) | ✅ Complete | CA policies, MFA, break-glass account, report-only → on, Zero Trust |
| 02 | [Privileged Identity Management (PIM)](lab-02-pim/) | ✅ Complete | Just-in-time access, eligible roles, approval workflow, audit logs |
| 03 | [Identity Protection & risky sign-ins](lab-03-identity-protection/) | ✅ Complete | Sign-in risk, user risk, Tor simulation, risk-based CA policies |
| 04 | [SSPR & authentication methods](lab-04-sspr-auth-methods/) | ✅ Complete | Self-service password reset, MFA methods, authentication methods policy |
| 05 | [Access Reviews](lab-05-access-reviews/) | ✅ Complete | Periodic governance, decision helpers, auto-apply, identity governance |
| 06 | [Access Packages (Entitlement Management)](lab-06-access-packages/) | ✅ Complete | Catalogs, access packages, request/approval workflow, auto-provisioning, time-bound access |
| 07 | [PIM for Groups](lab-07-pim-for-groups/) | ✅ Complete | Just-in-time group membership, role-assignable groups, activation workflow |
| 08 | [App Registration + OAuth2/OIDC](lab-08-app-registration-oauth2/) | ✅ Complete | App registration, delegated vs application permissions, admin consent, client secrets, OAuth2 endpoints |
| 09 | [Microsoft Graph API](lab-09-graph-api/) | ✅ Complete | REST API queries, OData parameters, permission consent flow, programmatic IAM verification |

> Each lab folder contains a detailed write-up with objectives, step-by-step configuration, screenshots, design decisions, lessons learned, and SC-300 relevant concepts. These are original labs — not copied from courses — built to document real IAM decisions in an enterprise-style tenant.

---

## Zero Trust Mapping

Every lab maps to one or more Zero Trust pillars (NIST 800-207 / Microsoft Zero Trust model):

| Pillar | Labs |
|---|---|
| **Verify explicitly** | 01 (CA/MFA), 03 (risk-based auth), 08 (OAuth2 app identity), 09 (API permission scopes) |
| **Least privilege access** | 02 (PIM JIT roles), 06 (time-bound access packages), 07 (PIM JIT group membership) |
| **Assume breach** | 03 (Identity Protection risk detection), 04 (SSPR self-remediation), 05 (periodic access reviews) |

---

## IAM Lifecycle Coverage

This lab series covers the complete identity lifecycle that an IAM Engineer manages in production:

```
┌─────────────────────────────────────────────────────────────────┐
│                    IDENTITY LIFECYCLE                           │
│                                                                 │
│  Authentication        Governance           Automation          │
│  ┌──────────┐         ┌──────────┐         ┌──────────┐         │
│  │ Lab 01   │         │ Lab 05   │         │ Lab 09   │         │
│  │ Cond.    │         │ Access   │         │ Graph    │         │
│  │ Access   │         │ Reviews  │         │ API      │         │
│  └──────────┘         └──────────┘         └──────────┘         │
│  ┌──────────┐         ┌──────────┐         ┌──────────┐         │
│  │ Lab 03   │         │ Lab 06   │         │ Lab 08   │         │
│  │ Identity │         │ Access   │         │ App Reg  │         │
│  │ Protect. │         │ Packages │         │ OAuth2   │         │
│  └──────────┘         └──────────┘         └──────────┘         │
│  ┌──────────┐         ┌──────────┐                              │
│  │ Lab 04   │         │ Lab 02   │                              │
│  │ SSPR     │         │ PIM      │                              │
│  └──────────┘         │ (roles)  │                              │
│                       └──────────┘                              │
│                       ┌──────────┐                              │
│                       │ Lab 07   │                              │
│                       │ PIM      │                              │
│                       │ (groups) │                              │
│                       └──────────┘                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Tenant Architecture

```
DidacIAMLab.onmicrosoft.com
├── Users
│   ├── DidacSanchez (Global Admin)
│   ├── testpim (test user — MFA via Authenticator + SMS)
│   └── breakglass (emergency access, permanent GA, no MFA, excluded from CA)
├── Groups
│   ├── SG-AllUsers (standard users subject to CA policies)
│   ├── SG-CA-Excluded (break-glass exclusion from all CA)
│   ├── SG-Finance-App-Access (access package resource — Lab 06)
│   └── SG-Privileged-App-Admins (PIM-managed group — Lab 07)
├── App Registrations
│   └── IAM-Lab-WebApp (Lab 08 — OAuth2/OIDC, single tenant)
├── Licenses
│   └── Microsoft Entra ID P2 (trial — labs built Jul–Aug 2026)
├── Conditional Access Policies (3 active)
│   ├── CA-001: Require MFA — All users
│   ├── CA-002: Sign-in risk — Require MFA (Medium+High)
│   └── CA-003: User risk — Require password change (Medium+High)
├── Identity Governance
│   ├── Catalog: IAM-Lab-Catalog
│   ├── Access Package: AP-Finance-Access (30-day expiry, approval required)
│   └── Access Review: Quarterly review — SG-AllUsers
├── PIM
│   ├── User Administrator — testpim eligible (4h, MFA, approval)
│   └── SG-Privileged-App-Admins — testpim eligible member (2h, MFA, approval)
└── Security Defaults: Disabled (migrated to Conditional Access)
```

---

## Documented Patterns

Recurring patterns identified and documented across multiple labs:

- **Microsoft is centralizing identity controls**: risk policies migrating from Identity Protection to Conditional Access (Oct 2026); SSPR methods merging into the unified Authentication Methods policy. Labs 03 and 04 document this transition.
- **Portal errors are often cosmetic**: PIM "Activate role failed — Unknown error" (Lab 02) and "The resource is invalid" (Lab 07) did not block functionality. Documented as lessons learned.
- **Permission consent is per-application, not per-tenant**: Admin consent granted on IAM-Lab-WebApp (Lab 08) did not carry to Graph Explorer (Lab 09). Each app maintains isolated permission grants.
- **Troubleshooting adds credibility**: Every lab documents real issues encountered — from SMS not enabled in authentication methods (Lab 04) to 403 errors requiring explicit consent (Lab 09).

---

## Why This Repo Exists

Most IAM candidates have certifications but no visible proof of hands-on work. This repo bridges that gap — every configuration, test and troubleshooting session happened in my own tenant, with real screenshots and real errors.

Built over several weeks alongside a full-time support job. I used AI assistance to structure and polish the written documentation; the tenant design, the configuration decisions, the testing and the problem-solving are mine. Every screenshot comes from DidacIAMLab.onmicrosoft.com.

The design decisions explain *why*, not just *how* — because in identity work, the reasoning matters more than the click path.

---

## Connect

- **LinkedIn**: [linkedin.com/in/didac-sf](https://linkedin.com/in/didac-sf)
- **Portfolio**: [didac-sf-portfolio.netlify.app](https://didac-sf-portfolio.netlify.app)
- **Email**: didac.sanchez.fontanet@gmail.com

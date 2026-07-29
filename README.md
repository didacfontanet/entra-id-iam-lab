# Microsoft Entra ID — Identity & Access Management Labs

Hands-on IAM labs built in a dedicated Microsoft Entra ID tenant, documenting real configurations, design decisions, and audit verification for every identity scenario.

## About

| | |
|---|---|
| **Author** | Didac Sanchez Fontanet |
| **Tenant** | DidacIAMLab.onmicrosoft.com |
| **License** | Entra ID P2 (trial) + Microsoft 365 Business Basic |
| **Stack** | Microsoft Entra ID · Conditional Access · PIM · Identity Protection · RBAC |
| **Certifications** | SC-900 (exam: 8 Aug 2026) → SC-300 → AZ-500 |

## Labs

| # | Lab | Status | Key topics |
|---|-----|--------|------------|
| 01 | [Conditional Access — MFA policy from scratch](./lab-01-conditional-access/) | ✅ Complete | CA policies, MFA, break-glass account, Zero Trust |
| 02 | [Privileged Identity Management (PIM)](./lab-02-pim/) | ✅ Complete | Just-in-time access, eligible roles, approval workflow, audit logs |
| 03 | [Identity Protection & risky sign-ins](./lab-03-identity-protection/) | 🔄 Next up | User risk policy, sign-in risk policy, risk remediation |
| 04 | [SSPR & authentication methods](./lab-04-sspr-auth-methods/) | ⬜ Planned | Self-service password reset, MFA methods, user experience |
| 05 | [Access Reviews & entitlement management](./lab-05-access-reviews/) | ⬜ Planned | Periodic access reviews, access packages, identity governance |

> **Note**: Each lab folder contains a detailed write-up with objectives, step-by-step configuration, screenshots, and lessons learned. These are original labs — not copied from courses — built to document real IAM decisions in an enterprise-style tenant.

## Tenant architecture

```
DidacIAMLab.onmicrosoft.com
├── Users
│   ├── DidacSanchez (Global Admin)
│   ├── testpim (test user)
│   └── breakglass (emergency access, permanent GA)
├── Groups
│   ├── SG-AllUsers (users subject to CA policies)
│   └── SG-CA-Excluded (break-glass exclusion group)
├── Licenses
│   └── Microsoft Entra ID P2
├── Policies
│   └── CA-001: Require MFA - All users (On)
└── Security Defaults: Disabled (migrated to Conditional Access)
```

## Why this repo exists

Most IAM candidates have certifications but no visible proof of hands-on work. This repo bridges that gap — every lab is something I configured, tested, and documented myself, with real screenshots from my own tenant.

## Connect

- [LinkedIn](https://www.linkedin.com/in/didac-sf/)
- [Portfolio](https://didac-sf-portfolio.netlify.app)

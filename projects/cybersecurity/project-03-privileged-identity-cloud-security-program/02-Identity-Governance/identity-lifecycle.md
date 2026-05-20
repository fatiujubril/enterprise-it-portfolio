# Identity Lifecycle Management
## FatiuLab Security — Joiner / Mover / Leaver Program

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Regulatory Context:** PCI-DSS, PIPEDA  
**Tenant:** FatiuLabSecurity.onmicrosoft.com  
**Policy Author:** Global Administrator  
**Last Reviewed:** May 2026  
**Status:** Active — Phase 1 Complete

---

## 1. Overview

Identity lifecycle management defines how user accounts and access rights are created, modified, and removed as employees join, change roles, and leave FatiuLab Security. It is the operational process that keeps the identity environment clean, compliant, and secure over time.

Without a defined lifecycle process, identity environments degrade rapidly:

```
WITHOUT lifecycle management:        WITH lifecycle management:
New hire gets broad access      →    New hire gets role-appropriate access
Role changes — old access stays      Access reviewed and updated on role change
Employee leaves — account stays      Account disabled and access removed promptly
                                     
Result after 12 months:              Result after 12 months:
Ghost accounts with active access    Clean identity environment
Privilege accumulation               Least privilege maintained
Compliance violations                Audit-ready evidence
Increased attack surface             Reduced attack surface
```

In a PCI-DSS regulated FinTech, identity lifecycle failures are a direct compliance risk. PCI-DSS Requirement 8 mandates that user accounts are managed throughout their lifecycle — from creation through modification to termination.

---

## 2. User Population

| User Type | Account | Groups | PIM Eligible | CA Policy |
|---|---|---|---|---|
| Global Administrator | globaladmin@ | SEC-Global-Admins | Permanent active GA | CA01, CA02, CA04 |
| Security Administrator | securityadmin@ | SEC-Admins, SEC-PIM-Eligible | GA + Security Admin (eligible) | CA01, CA02, CA04 |
| Standard User | standarduser@ | SEC-Standard-Users | None | CA01, CA05 (report-only) |
| Break-Glass | breakglass@ | SEC-Global-Admins, SEC-CA-Exclusions | Permanent active GA | CA03 only |

---

## 3. Joiner Process

The joiner process defines how new employees are onboarded into the identity environment with appropriate access from day one — no more, no less.

### 3.1 Standard User Onboarding

```
Day 0 — Pre-provisioning (IT/HR)
├── Account created in Entra ID
├── License assigned (M365 Business Premium)
├── Added to SEC-Standard-Users group
├── CA05 applies in report-only mode
└── Welcome email sent with temporary password

Day 1 — MFA Registration (Employee)
├── Employee signs in with temporary password
├── Forced password change on first login
├── MFA registration completed (Microsoft Authenticator)
├── MFA registration confirmed by IT
└── Employee added to role-specific groups as required

Day 1+ — Access Validation
├── Employee confirms access to required applications
├── IT confirms MFA is registered and working
└── Onboarding ticket closed
```

**MFA registration is mandatory before access is granted to sensitive systems.** Standard users remain in the report-only CA05 state until MFA registration is confirmed — at which point CA05 enforcement applies.

**Target state — staged group model:**

```
New employee created
        ↓
Added to SEC-New-Users (onboarding group)
        ↓
MFA registration completed on Day 1
        ↓
Moved from SEC-New-Users → SEC-Standard-Users
        ↓
CA05 enforces MFA — user already registered, no lockout risk
```

This staged model ensures MFA is always registered before enforcement applies. Currently implemented as a manual process — automation via Entra ID Lifecycle Workflows is a planned improvement.

### 3.2 Privileged User Onboarding

Privileged users (Security Administrators) follow the standard user process plus additional steps:

```
Standard onboarding steps (above) PLUS:

├── Added to SEC-Admins group
├── Added to SEC-PIM-Eligible group
├── PIM eligible assignment created:
│   └── Security Administrator role — 6 month expiry
├── PIM role settings reviewed with new admin
├── JIT activation workflow demonstrated
├── Break-glass procedure documented and shared
└── First access review scheduled
```

Privileged users are never given permanent active role assignments. All privileged access is JIT from day one — there is no grace period of standing privilege during onboarding.

### 3.3 Account Provisioning Standards

| Standard | Requirement |
|---|---|
| Naming convention | firstname.lastname@FatiuLabSecurity.onmicrosoft.com |
| Password policy | Minimum 14 characters, complexity required |
| MFA | Required before access to any application |
| License | M365 Business Premium + Entra ID P2 |
| Default groups | Role-appropriate groups only — no broad default access |
| Privileged access | JIT via PIM only — no standing privilege |

---

## 4. Mover Process

The mover process defines how access is updated when an employee changes roles, departments, or responsibilities. Role changes are the most common source of privilege accumulation — access from old roles is rarely removed when new access is granted.

### 4.1 Standard Role Change

```
Role change approved (manager + HR)
        ↓
IT notified via change request
        ↓
New access granted:
├── Added to new role-appropriate groups
├── New application access provisioned
└── PIM eligibility updated if required
        ↓
Old access removed:
├── Removed from previous role groups
├── Previous PIM eligibility removed if no longer needed
└── Access review triggered for affected groups
        ↓
Change documented and ticket closed
```

**Critical principle — remove before grant is not required, but remove must happen:**

New access can be granted immediately to avoid business disruption. However old access removal must occur within 5 business days of the role change. This is not optional — PCI-DSS Requirement 7.2 requires that access privileges are updated when job responsibilities change.

### 4.2 Privilege Escalation (Promotion to Admin Role)

When a standard user is promoted to a role requiring privileged access:

```
Promotion approved
        ↓
Standard user account reviewed — existing access appropriate?
        ↓
Security Admin onboarding process followed (Section 3.2)
        ↓
Standard user group membership reviewed:
└── Remain in SEC-Standard-Users? Or remove?
    (decision based on whether standard user access still needed)
        ↓
PIM eligibility assigned with appropriate expiry
        ↓
Admin training completed:
├── JIT activation workflow
├── Break-glass procedure
├── CA policy awareness
└── PIM audit trail responsibilities
        ↓
Access review triggered for all affected groups
```

### 4.3 Privilege Reduction (Demotion from Admin Role)

When an employee moves from a privileged role to a standard role:

```
Role change approved
        ↓
All active PIM role activations deactivated immediately
        ↓
PIM eligible assignments removed
        ↓
Removed from privileged groups:
├── SEC-Admins
├── SEC-PIM-Eligible
└── SEC-Global-Admins (if applicable)
        ↓
Added to appropriate standard user groups
        ↓
Access review triggered for all affected groups
        ↓
Change documented — audit trail maintained
```

**Privilege reduction is treated as a security-sensitive operation** — it must be completed the same day as the role change, not deferred.

---

## 5. Leaver Process

The leaver process defines how access is removed when an employee leaves FatiuLab Security. Delayed account termination is one of the most common identity security failures — former employees retaining active access to systems they no longer have a business need to access.

### 5.1 Planned Departure (Resignation / End of Contract)

```
Departure confirmed (HR notifies IT)
        ↓
Last working day — End of Day:
├── Account disabled in Entra ID
├── All active sessions revoked (Sign out everywhere)
├── All active PIM role activations deactivated
├── MFA methods reviewed (for audit purposes)
└── Manager notified of completion
        ↓
Day 1 after departure:
├── Removed from all security groups
├── PIM eligible assignments removed
├── License deallocated
└── Mailbox converted to shared mailbox or deleted per policy
        ↓
30 days after departure:
├── Account permanently deleted
├── All group memberships confirmed removed
└── Leaver ticket closed with audit trail
```

### 5.2 Unplanned Departure (Termination)

Unplanned departures require immediate action — the risk of a disgruntled former employee retaining access is significantly higher than planned departures.

```
Termination decision made
        ↓
IMMEDIATE (same hour):
├── Account disabled in Entra ID
├── All active sessions revoked
├── All active PIM role activations deactivated
├── Passwords reset (prevents re-enable attacks)
└── Manager and Security Admin notified
        ↓
Within 24 hours:
├── All group memberships removed
├── PIM eligible assignments removed
├── License deallocated
├── Access review of all systems accessed in last 30 days
└── Audit log reviewed for any suspicious activity before termination
        ↓
Within 7 days:
└── Full access review completed and documented
```

### 5.3 Account Termination Standards

| Standard | Requirement |
|---|---|
| Disable timeline — planned | End of last working day |
| Disable timeline — unplanned | Immediate (same hour) |
| Session revocation | Immediate on account disable |
| Group removal | Within 24 hours of disable |
| License deallocation | Within 24 hours of disable |
| Account deletion | 30 days after departure |
| Audit log review | Within 7 days for all departures |

---

## 6. Privileged Account Lifecycle

Privileged accounts follow a stricter lifecycle than standard accounts due to the elevated risk they represent.

### 6.1 Break-Glass Account

The break-glass account is a special case — it is never disabled, never has MFA registered, and is permanently excluded from CA policies. Its lifecycle is governed differently:

| Control | Standard | Break-Glass |
|---|---|---|
| MFA | Required | Never registered — intentional |
| CA policy enforcement | Full enforcement | Excluded via SEC-CA-Exclusions |
| Account disable on departure | Yes | Never disabled |
| Password rotation | On compromise | Quarterly rotation |
| Access review | Standard | Monthly via AR-02 |
| Usage monitoring | Standard logging | Any use = security event |

**Break-glass credential management:**
- Credentials stored offline in secure physical location
- Password rotated quarterly and on any use
- Access to credentials requires dual authorization
- Every use documented as a security incident — root cause analysis required

### 6.2 PIM Eligible Assignment Lifecycle

PIM eligible assignments have their own lifecycle — they are not permanent and must be renewed:

```
Eligible assignment created (onboarding or mover)
        ↓
Assignment active for defined period:
├── Global Administrator — 3 months
└── Security Administrator — 6 months
        ↓
Expiry approaching — access review triggered (AR-01)
        ↓
Reviewer decision:
├── Still needed → renew assignment for another period
└── No longer needed → assignment expires, not renewed
        ↓
If not renewed — user loses eligibility automatically
No manual removal required
```

---

## 7. Lifecycle and Access Review Integration

Access Reviews are the automated governance check on the lifecycle process — they catch cases where the manual process was not followed correctly:

| Lifecycle Failure | Access Review Catch |
|---|---|
| Leaver account not removed from groups | AR-01 flags inactive account (30+ day no sign-in helper) |
| Mover — old group membership not removed | AR-01 quarterly review catches residual access |
| Unauthorized addition to CA exclusion group | AR-02 monthly review catches within 30 days |
| PIM eligibility not removed after role change | AR-01 quarterly review covers PIM-eligible groups |

Access Reviews are the safety net — they assume the manual process will sometimes fail and catch the failures before they become security incidents.

---

## 8. Gaps and Planned Improvements

| Gap | Risk | Planned Remediation | Priority |
|---|---|---|---|
| Manual onboarding process | Human error in provisioning | Implement Entra ID Lifecycle Workflows for automation | High |
| No HR system integration | Leavers may not be reported to IT promptly | Integrate HR system with Entra ID for automated disable | High |
| No automated deprovisioning | Manual group removal on offboarding | Lifecycle Workflows automation | High |
| No privileged access recertification workflow | PIM renewals handled manually | Automate via access review triggered renewal | Medium |
| No access certification for application roles | App-level access not reviewed | Extend access reviews to cover application assignments | Medium |
| SEC-New-Users group not yet created | Staged onboarding model not yet implemented | Create SEC-New-Users group and onboarding CA policy | Medium |

---

## 9. Compliance Alignment

| Control | PCI-DSS | PIPEDA | NIST CSF | Zero Trust |
|---|---|---|---|---|
| Account provisioning standards | Req 8.2 | Principle 4 | PR.AC-1 | Least privilege |
| MFA required before access | Req 8.4 | Principle 7 | PR.AC-7 | Verify explicitly |
| Role-appropriate access only | Req 7.2 | Principle 4 | PR.AC-4 | Least privilege |
| Access updated on role change | Req 7.2.4 | Principle 4 | PR.AC-1 | Least privilege |
| Immediate disable on termination | Req 8.3.4 | Principle 4 | PR.AC-1 | Assume breach |
| 30-day account deletion | Req 8.3.4 | Principle 4 | PR.AC-1 | Least privilege |
| Audit trail on all changes | Req 10.2 | Principle 4 | DE.CM-3 | Assume breach |
| Break-glass credential management | Req 8.5 | Principle 7 | PR.AC-4 | Verify explicitly |

---

## 10. Evidence Index

Identity lifecycle management is a process document — the evidence for this section is found across the baseline, PIM, and access review evidence:

| File | Description |
|---|---|
| [P3-BL-01-Tenant-Overview.png](../Evidence/Phase-01-Identity/P3-BL-01-Tenant-Overview.png) | Tenant foundation |
| [P3-BL-02-P2-License-Assigned.png](../Evidence/Phase-01-Identity/P3-BL-02-P2-License-Assigned.png) | License provisioning |
| [P3-BL-03-Users-List.png](../Evidence/Phase-01-Identity/P3-BL-03-Users-List.png) | User accounts — all lifecycle stages represented |
| [P3-BL-04-Security-Groups.png](../Evidence/Phase-01-Identity/P3-BL-04-Security-Groups.png) | Security groups — role-based access model |
| [P3-PIM-03-Eligible-Assignments.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03-Eligible-Assignments.png) | PIM eligible assignments with expiry dates |
| [P3-PIM-03d-Active-Assignments.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03d-Active-Assignments.png) | Active assignments — Global Admin only |
| [P3-AR-00-Access-Reviews-List.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-00-Access-Reviews-List.png) | Access reviews — lifecycle safety net |
| [P3-AR-01-Privileged-Groups-Review-Settings.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-01-Privileged-Groups-Review-Settings.png) | Quarterly privileged group review |
| [P3-AR-02-CA-Exclusions-Review-Settings.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-02-CA-Exclusions-Review-Settings.png) | Monthly CA exclusions review |

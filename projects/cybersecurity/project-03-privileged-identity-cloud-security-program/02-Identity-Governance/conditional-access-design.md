# Conditional Access Design
## FatiuLab Security — Zero Trust Identity Policy Framework

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Regulatory Context:** PCI-DSS, PIPEDA (Canadian Personal Information Protection and Electronic Documents Act)  
**Tenant:** FatiuLabSecurity.onmicrosoft.com  
**Policy Author:** Global Administrator  
**Last Reviewed:** May 2026  
**Status:** Active — Phase 1 Complete

---

## 1. Overview

FatiuLab Security is a 200-person financial technology company processing customer payment data and handling regulated financial information. As a FinTech operating under PCI-DSS and PIPEDA, protecting identity is the primary attack surface — compromised credentials are the leading vector for unauthorized access to payment infrastructure and customer data.

This document defines the Conditional Access policy framework implemented as part of the Privileged Identity & Cloud Security Program. All policies are designed around Zero Trust principles:

> **Never trust, always verify. Assume breach. Use least-privilege access.**

Conditional Access is the enforcement layer that translates Zero Trust principles into technical controls — every access request is evaluated against policy before being granted.

---

## 2. Design Principles

| Principle | Implementation |
|---|---|
| Verify explicitly | MFA required for all users on every authentication |
| Least privilege access | Standard users scoped to approved apps only |
| Assume breach | Legacy authentication blocked — no fallback to weak protocols |
| Protect privileged access | Stricter controls for admin roles accessing any resource |
| Emergency access preserved | Break-glass account excluded from all enforcement policies |
| Audit everything | All policies generate sign-in logs for SIEM ingestion |

---

## 3. Environment Prerequisites

### 3.1 Licensing
- Microsoft Entra ID P2 (included in Microsoft 365 Business Premium)
- Required for: Conditional Access, Identity Protection, PIM integration

Evidence: [P3-BL-02-P2-License-Assigned.png](../Evidence/Phase-01-Identity/P3-BL-02-P2-License-Assigned.png)

### 3.2 Security Groups

| Group | Purpose | Members |
|---|---|---|
| SEC-CA-Exclusions | CA policy exclusion — break-glass only | breakglass@ |
| SEC-Standard-Users | Standard user population for CA05 scoping | standarduser@ |
| SEC-Admins | Security administrator group | securityadmin@ |
| SEC-Global-Admins | Global administrator group | globaladmin@, breakglass@ |

Evidence: [P3-BL-04-Security-Groups.png](../Evidence/Phase-01-Identity/P3-BL-04-Security-Groups.png)

### 3.3 Named Locations

| Location | Type | IP Range | Trusted |
|---|---|---|---|
| LOC-FatiuLab-HQ | IP ranges | 192.168.1.0/24 | Yes |

Named location defined as the trusted corporate network perimeter. Referenced in policy maturity roadmap for step-up authentication enforcement outside trusted network boundaries.

Evidence: [P3-CA-07-Named-Location.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-07-Named-Location.png)

### 3.4 Break-Glass Account
- **Account:** breakglass@FatiuLabSecurity.onmicrosoft.com
- **Role:** Global Administrator (permanent active)
- **MFA:** Not registered — intentional emergency design
- **CA Exclusion:** Member of SEC-CA-Exclusions — excluded from all enforcement policies
- **Credential storage:** Offline, securely stored, documented exception
- **Monitoring:** Monitored via AR-02 monthly access review of SEC-CA-Exclusions group
- **Rationale:** Ensures tenant recovery capability if primary admin accounts or MFA systems become unavailable

Evidence: [P3-BL-03-Users-List.png](../Evidence/Phase-01-Identity/P3-BL-03-Users-List.png)

---

## 4. Policy Inventory

| Policy ID | Name | State | Scope | Control |
|---|---|---|---|---|
| CA01 | Require-MFA-All-Users | On | All users | Require MFA |
| CA02 | Require-MFA-Admins | On | Admin directory roles | Require MFA |
| CA03 | Block-Legacy-Authentication | On | All users | Block access |
| CA04 | Require-MFA-Azure-Management | On | All users | Require MFA |
| CA05 | Restrict-Standard-Users | Report-only | SEC-Standard-Users | Require MFA |

Evidence: [P3-CA-00-Policy-List.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-00-Policy-List.png)

---

## 5. Policy Specifications

### CA01 — Require MFA for All Users

**Purpose:** Baseline MFA enforcement across the entire user population. Ensures no user can authenticate to any cloud application without completing a second factor — the foundational Zero Trust control.

**Business justification:** PCI-DSS Requirement 8.4 mandates MFA for all access to the cardholder data environment. PIPEDA requires reasonable safeguards for personal information. Blanket MFA is the minimum acceptable control for a FinTech handling payment data.

| Setting | Value |
|---|---|
| Users — Include | All users |
| Users — Exclude | SEC-CA-Exclusions |
| Cloud apps | All cloud apps |
| Conditions | None |
| Grant | Require multifactor authentication |
| State | On |

**Design notes:**
- Exclusion of SEC-CA-Exclusions ensures break-glass account retains access if MFA infrastructure fails
- No device compliance requirement at this tier — applied at CA02 for privileged accounts
- Covers all cloud apps to prevent MFA bypass through lesser-known application access paths

**Evidence:**
- [P3-CA-01-Require-MFA-All-Users.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-01-Require-MFA-All-Users.png) — policy overview + grant control
- [P3-CA-01b-Require-MFA-All-Users-Include.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-01b-Require-MFA-All-Users-Include.png) — include scope
- [P3-CA-01c-Require-MFA-All-Users-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-01c-Require-MFA-All-Users-Exclude.png) — exclusion group

---

### CA02 — Require MFA for Admins

**Purpose:** Elevated enforcement for privileged directory roles. Admin accounts have access to identity infrastructure, security configuration, and payment system integrations — a compromised admin account represents catastrophic risk.

**Business justification:** PCI-DSS Requirement 7 (restrict access to system components) and Requirement 8 (identify users and authenticate access) specifically address privileged account controls. Admin MFA is non-negotiable in a payment processing environment.

| Setting | Value |
|---|---|
| Users — Include | Directory roles: Global Administrator, Security Administrator, Privileged Role Administrator |
| Users — Exclude | SEC-CA-Exclusions |
| Cloud apps | All cloud apps |
| Conditions | None |
| Grant | Require multifactor authentication |
| State | On |

**Design notes:**
- Targets three highest-risk directory roles — Global Admin, Security Admin, Privileged Role Admin
- Applies across all cloud apps — admin accounts should always MFA regardless of application
- Complements PIM controls: PIM governs when admins can activate roles; CA02 governs how they authenticate when they do

**Evidence:**
- [P3-CA-02-Require-MFA-Admins.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02-Require-MFA-Admins.png) — policy overview + grant control
- [P3-CA-02b-Require-MFA-Admins-Include-01.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02b-Require-MFA-Admins-Include-01.png) — directory roles selected (scroll 1)
- [P3-CA-02b-Require-MFA-Admins-Include-02.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02b-Require-MFA-Admins-Include-02.png) — directory roles selected (scroll 2)
- [P3-CA-02c-Require-MFA-Admins-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02c-Require-MFA-Admins-Exclude.png) — exclusion group

---

### CA03 — Block Legacy Authentication

**Purpose:** Eliminate legacy authentication protocols (Exchange ActiveSync, IMAP, POP3, SMTP AUTH, older Office clients) that cannot participate in MFA challenges. Legacy auth is the most common mechanism used to bypass MFA entirely.

**Business justification:** Microsoft telemetry shows over 99% of password spray attacks and a significant proportion of credential stuffing attacks use legacy authentication protocols. Blocking legacy auth eliminates this attack class entirely. PCI-DSS Requirement 6.2 requires that all system components are protected from known vulnerabilities — legacy auth protocols are a known, documented vulnerability.

| Setting | Value |
|---|---|
| Users — Include | All users |
| Users — Exclude | None |
| Cloud apps | All cloud apps |
| Conditions — Client apps | Exchange ActiveSync clients + Other clients |
| Grant | Block access |
| State | On |

**Design notes:**
- No exclusions — legacy auth blocking should be universal with no exceptions
- Break-glass account is not excluded here intentionally: break-glass should use modern authentication
- "Other clients" covers all legacy protocol clients beyond Exchange ActiveSync
- This policy has zero legitimate business exceptions in a modern Microsoft 365 environment

**Evidence:**
- [P3-CA-03-Block-Legacy-Auth.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-03-Block-Legacy-Auth.png) — policy overview + block grant
- [P3-CA-03b-Block-Legacy-Auth-Conditions.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-03b-Block-Legacy-Auth-Conditions.png) — client apps conditions
- [P3-CA-03c-Block-Legacy-Auth-Users.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-03c-Block-Legacy-Auth-Users.png) — all users scope

---

### CA04 — Require MFA for Azure Management

**Purpose:** Enforce MFA specifically for access to Azure Resource Manager (Azure portal, Azure CLI, Azure PowerShell, Azure SDKs). Protects cloud infrastructure from unauthorized modification even if a user's primary session is already authenticated.

**Business justification:** Azure Management access represents the highest-privilege action surface in the cloud environment — resource creation, security configuration, network changes, and IAM modifications all flow through this plane. A separate, dedicated MFA requirement for Azure Management ensures that even an attacker who has hijacked an authenticated session cannot make infrastructure changes without a fresh MFA challenge.

| Setting | Value |
|---|---|
| Users — Include | All users |
| Users — Exclude | SEC-CA-Exclusions |
| Cloud apps | Azure Resource Manager (App ID: 797f4846-ba00-4fd7-ba43-dac1f8f63013) |
| Conditions | None |
| Grant | Require multifactor authentication |
| State | On |

**Design notes:**
- Targets Azure Resource Manager specifically — not all cloud apps — to avoid redundancy with CA01
- App registered as "Azure Resource Manager" in this tenant (formerly "Microsoft Azure Management")
- Complements Defender for Cloud CSPM — CA04 prevents unauthorized changes; CSPM detects misconfigurations
- Works in conjunction with PIM: PIM requires MFA on role activation; CA04 requires MFA on Azure portal access

**Evidence:**
- [P3-CA-04-Require-MFA-Azure-Management.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04-Require-MFA-Azure-Management.png) — policy overview + grant
- [P3-CA-04b-Require-MFA-Azure-Management-Include.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04b-Require-MFA-Azure-Management-Include.png) — all users include
- [P3-CA-04c-Require-MFA-Azure-Management-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04c-Require-MFA-Azure-Management-Exclude.png) — exclusion group
- [P3-CA-04d-Require-MFA-Azure-Management-App.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04d-Require-MFA-Azure-Management-App.png) — Azure Resource Manager app selected

---

### CA05 — Restrict Standard Users (Report-only)

**Purpose:** Enforce MFA for all standard user population access. Currently in Report-only mode pending completion of the MFA registration campaign for the SEC-Standard-Users group.

**Business justification:** Standard users access M365 applications including Exchange Online, SharePoint, and Teams — all of which handle internal communications and potentially sensitive payment operations data. MFA enforcement for standard users is required for PCI-DSS scope compliance.

| Setting | Value |
|---|---|
| Users — Include | SEC-Standard-Users group |
| Users — Exclude | SEC-CA-Exclusions |
| Cloud apps | All cloud apps |
| Conditions | None |
| Grant | Require multifactor authentication |
| State | Report-only |

**Current state — Report-only rationale:**
CA05 is in Report-only mode because standard users in the SEC-Standard-Users group have not yet completed MFA registration. Enabling enforcement before registration is complete would lock users out of their accounts — an availability incident with direct business impact.

**Target state — Production maturity model:**

In a production enterprise environment, the onboarding workflow would be:

```
New user created
        ↓
Added to SEC-New-Users group
        ↓
MFA registration completed during onboarding (day 1)
        ↓
User moved to SEC-Standard-Users group
        ↓
CA05 enforces MFA — user already registered, no disruption
```

This staged group model ensures CA05 is always enforced on users who have MFA registered, while new users complete registration before enforcement applies. CA05 will be moved to **On** upon completion of the MFA registration campaign.

**Report-only monitoring:** Sign-in logs are reviewed during the report-only period to identify any users who would be blocked, validate policy logic, and confirm zero false positives before enforcement.

**Evidence:**
- [P3-CA-05-Restrict-Standard-Users.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-05-Restrict-Standard-Users.png) — policy overview
- [P3-CA-05b-Restrict-Standard-Users-Include.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-05b-Restrict-Standard-Users-Include.png) — SEC-Standard-Users group
- [P3-CA-05c-Restrict-Standard-Users-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-05c-Restrict-Standard-Users-Exclude.png) — exclusion group

---

## 6. Policy Interaction Matrix

| Scenario | CA01 | CA02 | CA03 | CA04 | CA05 | Result |
|---|---|---|---|---|---|---|
| Standard user → M365 apps | ✅ MFA | — | Block if legacy | — | Report-only | MFA required |
| Admin → M365 apps | ✅ MFA | ✅ MFA | Block if legacy | — | — | MFA required (both policies) |
| Admin → Azure portal | ✅ MFA | ✅ MFA | Block if legacy | ✅ MFA | — | MFA required (three policies) |
| Any user → Legacy client | — | — | ✅ Block | — | — | Blocked |
| Break-glass → Any app | Excluded | Excluded | Not excluded | Excluded | Excluded | Access granted — no MFA |

**Key insight:** Admin access to Azure portal triggers three simultaneous policy evaluations (CA01, CA02, CA04). All three require MFA — the most restrictive control applies. This is intentional defense-in-depth.

---

## 7. Named Locations

| Location | IP Range | Trusted | Used In |
|---|---|---|---|
| LOC-FatiuLab-HQ | 192.168.1.0/24 | Yes | Not yet assigned to policy |

Evidence: [P3-CA-07-Named-Location.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-07-Named-Location.png)

**Policy maturity roadmap — planned enhancement:**

A future policy `CA06-Require-Compliant-Device-Outside-Trusted-Location` will enforce device compliance for any access originating outside `LOC-FatiuLab-HQ`:

```
Access from LOC-FatiuLab-HQ (trusted) → MFA sufficient
Access from outside trusted network   → MFA + compliant device required
```

---

## 8. Gaps and Planned Improvements

| Gap | Risk | Planned Remediation | Priority |
|---|---|---|---|
| CA05 Report-only | Standard users not MFA-enforced | Complete MFA registration campaign → enable enforcement | High |
| No device compliance policy | Unmanaged devices can access resources with MFA only | Add Intune device compliance + CA06 | Medium |
| No location-based step-up auth | No differentiation between on-network and off-network access | Deploy CA06 using LOC-FatiuLab-HQ | Medium |
| No sign-in risk policy | Risky sign-ins not automatically challenged | Add Identity Protection risk-based CA policy | Medium |
| No session controls | No adaptive session lifetime management | Configure session controls for privileged roles | Low |

---

## 9. Compliance Alignment

| Control | PCI-DSS | PIPEDA | NIST CSF | Zero Trust |
|---|---|---|---|---|
| MFA all users (CA01) | Req 8.4 | Principle 7 | PR.AC-7 | Verify explicitly |
| MFA admins (CA02) | Req 8.4, 7.2 | Principle 7 | PR.AC-4 | Least privilege |
| Block legacy auth (CA03) | Req 6.2, 8.3 | Principle 7 | PR.PT-3 | Assume breach |
| MFA Azure management (CA04) | Req 7.2, 8.4 | Principle 7 | PR.AC-4 | Protect admin plane |
| Standard user MFA (CA05) | Req 8.4 | Principle 7 | PR.AC-7 | Verify explicitly |

---

## 10. Evidence Index

### Baseline
| File | Description |
|---|---|
| [P3-BL-01-Tenant-Overview.png](../Evidence/Phase-01-Identity/P3-BL-01-Tenant-Overview.png) | Tenant overview — FatiuLab Security |
| [P3-BL-02-P2-License-Assigned.png](../Evidence/Phase-01-Identity/P3-BL-02-P2-License-Assigned.png) | Entra ID P2 license assigned |
| [P3-BL-03-Users-List.png](../Evidence/Phase-01-Identity/P3-BL-03-Users-List.png) | All 4 users including break-glass |
| [P3-BL-04-Security-Groups.png](../Evidence/Phase-01-Identity/P3-BL-04-Security-Groups.png) | All 5 security groups |

### Conditional Access Policies
| File | Description |
|---|---|
| [P3-CA-00-Policy-List.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-00-Policy-List.png) | All 5 CA policies — names and states |
| [P3-CA-01-Require-MFA-All-Users.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-01-Require-MFA-All-Users.png) | CA01 overview + grant |
| [P3-CA-01b-Require-MFA-All-Users-Include.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-01b-Require-MFA-All-Users-Include.png) | CA01 include scope |
| [P3-CA-01c-Require-MFA-All-Users-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-01c-Require-MFA-All-Users-Exclude.png) | CA01 exclusion group |
| [P3-CA-02-Require-MFA-Admins.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02-Require-MFA-Admins.png) | CA02 overview + grant |
| [P3-CA-02b-Require-MFA-Admins-Include-01.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02b-Require-MFA-Admins-Include-01.png) | CA02 directory roles (scroll 1) |
| [P3-CA-02b-Require-MFA-Admins-Include-02.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02b-Require-MFA-Admins-Include-02.png) | CA02 directory roles (scroll 2) |
| [P3-CA-02c-Require-MFA-Admins-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02c-Require-MFA-Admins-Exclude.png) | CA02 exclusion group |
| [P3-CA-03-Block-Legacy-Auth.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-03-Block-Legacy-Auth.png) | CA03 overview + block grant |
| [P3-CA-03b-Block-Legacy-Auth-Conditions.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-03b-Block-Legacy-Auth-Conditions.png) | CA03 client apps conditions |
| [P3-CA-03c-Block-Legacy-Auth-Users.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-03c-Block-Legacy-Auth-Users.png) | CA03 all users scope |
| [P3-CA-04-Require-MFA-Azure-Management.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04-Require-MFA-Azure-Management.png) | CA04 overview + grant |
| [P3-CA-04b-Require-MFA-Azure-Management-Include.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04b-Require-MFA-Azure-Management-Include.png) | CA04 include scope |
| [P3-CA-04c-Require-MFA-Azure-Management-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04c-Require-MFA-Azure-Management-Exclude.png) | CA04 exclusion group |
| [P3-CA-04d-Require-MFA-Azure-Management-App.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04d-Require-MFA-Azure-Management-App.png) | CA04 Azure Resource Manager app |
| [P3-CA-05-Restrict-Standard-Users.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-05-Restrict-Standard-Users.png) | CA05 report-only overview |
| [P3-CA-05b-Restrict-Standard-Users-Include.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-05b-Restrict-Standard-Users-Include.png) | CA05 SEC-Standard-Users group |
| [P3-CA-05c-Restrict-Standard-Users-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-05c-Restrict-Standard-Users-Exclude.png) | CA05 exclusion group |
| [P3-CA-07-Named-Location.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-07-Named-Location.png) | LOC-FatiuLab-HQ trusted location |

# Conditional Access – Administrative Baseline

## Purpose

This document defines the **baseline Conditional Access (CA) controls** applied to administrative accounts within the tenant.

The objective is to:
- Enforce strong authentication for all administrative access
- Block legacy authentication paths commonly used in account compromise
- Preserve tenant recoverability through explicitly excluded break-glass accounts

These controls align with **Zero Trust**, **least privilege**, and Microsoft security best practices.

---

## Scope

**Targeted users:**
- Security administrators scoped via the `SEC-CA-Admins` security group

**Explicit exclusions:**
- Emergency access accounts contained in `SEC-BreakGlass`

**Targeted applications:**
- All cloud applications
- Microsoft 365 Admin Services

---

## Conditional Access Policies

---

### CA-01 – Require MFA for Admins

**Policy name:**  
`CA-01-Require-MFA-Admins`

**Purpose:**  
Ensures that all administrative sign-ins require multi-factor authentication, reducing the risk of credential compromise.

**Assignments:**
- **Include:** `SEC-CA-Admins`
- **Exclude:** `SEC-BreakGlass`

**Target resources:**
- All cloud apps

**Access controls:**
- Grant access
- Require multi-factor authentication

**Enforcement state:**
- Enabled (On)

**Evidence:**

![CA-01 enabled](./Evidence/Phase-01-Identity/CA-Policies/20260207_CA01_RequireMFA_Admins_Enabled.png)

---

### CA-02 – Block Legacy Authentication for Admins

**Policy name:**  
`CA-02-Block-LegacyAuth-Admins`

**Purpose:**  
Blocks legacy authentication protocols (e.g., basic authentication) that bypass modern security controls and are frequently abused in attacks.

**Assignments:**
- **Include:** `SEC-CA-Admins`
- **Exclude:** `SEC-BreakGlass`

**Target resources:**
- All cloud apps

**Conditions:**
- Client apps:
  - Exchange ActiveSync clients
  - Other legacy authentication clients

**Access controls:**
- Block access

**Enforcement state:**
- Enabled (On)

**Evidence:**

![CA-02 enabled](./Evidence/Phase-01-Identity/CA-Policies/20260207_CA02_BlockLegacyAuth_Admins_Enabled.png)

---

## Policy Validation

Prior to enforcement, both policies were evaluated using the **Conditional Access What If tool** to confirm:

- Correct policy targeting
- No unintended impact to break-glass accounts
- Proper evaluation against administrative cloud services

**Validation method:**
- User: `sec.admin`
- Application: Microsoft 365 Admin Services
- Result: Both policies applied as expected in report-only mode

**Evidence:**

![Conditional Access What If results](./Evidence/Phase-01-Identity/CA-Policies/20260207_CA_WhatIf_AdminPolicies.png)

---

## Security Rationale

- Administrative accounts are high-value targets and require stronger controls than standard users
- Legacy authentication bypasses MFA and modern security protections
- Break-glass accounts are excluded to ensure tenant recovery during Conditional Access or identity service failures

---

## Summary

These Conditional Access policies establish a strong administrative security baseline by:

- Enforcing MFA for all admin access
- Eliminating legacy authentication attack paths
- Preserving emergency access through controlled exclusions

This layer works in conjunction with **Privileged Identity Management (PIM)** to ensure that administrative access is **just-in-time, strongly authenticated, and auditable**.

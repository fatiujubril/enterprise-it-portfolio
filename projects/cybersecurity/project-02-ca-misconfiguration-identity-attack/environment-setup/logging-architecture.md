# Logging Architecture – Identity & Conditional Access

This document describes the logging and telemetry sources used in Project 2 to detect, investigate, and remediate Conditional Access misconfigurations within Microsoft Entra ID.

The focus is on **identity-layer visibility**, not endpoint or network telemetry.

---

## Logging Scope

The following Microsoft Entra ID logging sources were enabled and used throughout this project:

- **Sign-in Logs (Interactive)**
- **Sign-in Logs (Non-Interactive)**
- **Audit Logs**
- **Conditional Access Evaluation Results**

These logs provide sufficient visibility to detect authentication downgrades, policy exclusions, and MFA enforcement gaps.

---

## Primary Log Sources

### 1. Sign-in Logs (Interactive)

**Purpose**
- Identify successful and failed user authentications
- Validate Conditional Access enforcement
- Detect MFA bypass conditions

**Key Fields Used**
- User
- Application
- Status
- Conditional Access
- Authentication requirement
- Sign-in error code
- IP address
- Location
- Client / agent type

**Why This Matters**
Interactive sign-in logs are the primary signal used to detect that MFA was **not applied** to the CEO account despite an active MFA policy.

---

### 2. Conditional Access Evaluation

**Purpose**
- Determine whether Conditional Access policies were applied
- Identify exclusions or scope mismatches
- Confirm authentication strength requirements

**Key Indicators**
- Conditional Access result: `Success` vs `Not Applied`
- Authentication requirement: `Multifactor authentication` vs `Single-factor authentication`

**Why This Matters**
The absence of Conditional Access enforcement — rather than a failure — was the core detection indicator in this incident.

---

### 3. Audit Logs

**Purpose**
- Track administrative configuration changes
- Identify policy modifications and exclusions
- Establish timeline context for misconfiguration

**Relevant Activities**
- Conditional Access policy updates
- User or group exclusions
- Authentication method changes

**Why This Matters**
Audit logs provide accountability and explain *how* the misconfiguration was introduced, even when the action was non-malicious.

---

## Logging Gaps Observed

During the scenario, the following gaps were identified:

- No default alert for high-risk account MFA exclusion
- No automatic expiration or review for Conditional Access exceptions
- No alert when Conditional Access evaluates as `Not Applied` for privileged users

These gaps allowed the misconfiguration to persist without detection until manual review.

---

## Logging Design Assumptions

This project assumes:
- Cloud-native identity-only environment
- No dependency on endpoint detection tooling
- No SIEM ingestion (manual portal analysis)

This mirrors many small-to-mid-sized Microsoft 365 tenants where identity logs are reviewed directly in Entra ID.

---

## Transition to Detection Phase

The logging architecture described above enabled the detection of the Conditional Access MFA bypass through sign-in telemetry analysis.

Subsequent sections demonstrate how these logs were reviewed to identify, validate, and respond to the misconfiguration.

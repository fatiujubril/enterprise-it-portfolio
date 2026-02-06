# Attack Scenario – Conditional Access Exception Abuse

This section documents the simulated attack scenario used in Project 2.  
The objective is to demonstrate how a **legitimate administrative decision** can unintentionally weaken identity security controls and create a high-risk authentication bypass.

This scenario does **not** involve phishing, malware, or external threat actors.  
Instead, it focuses on **internal misconfiguration and exception handling failure** at the identity layer.

---

## Scenario Overview

The organization enforces multifactor authentication (MFA) using Microsoft Entra ID Conditional Access. MFA is required for all interactive user sign-ins as part of the baseline security posture.

During normal operations, the Chief Executive Officer (CEO) experienced an authentication disruption following a mobile device change affecting Microsoft Authenticator. MFA prompts could not be completed successfully, preventing access to business-critical Microsoft 365 services.

To restore access quickly, IT administrators temporarily excluded the CEO account from the Conditional Access policy enforcing MFA.

The exclusion was intended to be **temporary**, but no expiration, approval tracking, or follow-up review mechanism was implemented.

---

## Attack Timeline

**T0 – Secure Baseline Established**
- Conditional Access policy enforces MFA for all users
- CEO account successfully authenticates using MFA
- Sign-in logs confirm MFA enforcement is applied

**T1 – Authentication Disruption**
- CEO replaces or reconfigures mobile device
- Microsoft Authenticator registration becomes unavailable
- MFA challenges cannot be completed

**T2 – Emergency Access Request**
- CEO reports inability to access Microsoft 365 services
- Business impact identified as high priority
- IT approves temporary access workaround

**T3 – Conditional Access Exception Introduced**
- CEO account is explicitly excluded from the MFA Conditional Access policy
- Policy remains enabled for all other users
- No expiration or monitoring is configured for the exclusion

**T4 – Security Degradation**
- Conditional Access evaluates as **Not Applied** for CEO sign-ins
- Authentication requirement downgrades to **single-factor**
- Successful sign-ins occur without MFA enforcement

**T5 – Exposure Persists**
- No automated alerts are triggered
- Environment appears compliant at a policy level
- High-value account remains unprotected until manually reviewed

---

## Attack Condition Introduced

The following conditions were present during the scenario:

- The Conditional Access policy enforcing MFA remained enabled
- All standard users continued to be protected by MFA
- The CEO account was explicitly excluded from the policy
- The exclusion caused Conditional Access to evaluate as **Not Applied**
- The CEO authenticated using **single-factor authentication**

At this point, the environment entered a **high-risk identity state**, despite appearing secure at a policy configuration level.

---

## Why This Represents an Attack Path

Although no malicious activity occurred, the resulting condition is functionally equivalent to an identity attack:

- A high-value account could authenticate without MFA
- Authentication logs showed downgraded security requirements
- No default alerts indicated a loss of protection
- The exposure persisted silently until detection

This demonstrates how **Conditional Access exceptions can override security intent**, creating an attack surface without external compromise.

---

## Threat Classification

- **Attack Type:** Internal Conditional Access exception abuse  
- **Attack Vector:** Policy-based MFA exclusion  
- **Root Cause:** Inadequate exception governance  
- **Threat Actor:** Non-malicious internal action  
- **Impact:** Single-factor authentication exposure on a high-value identity  

---

## Transition to Detection Phase

The attack scenario concludes once the Conditional Access exclusion is active and the CEO account begins authenticating without MFA enforcement.

Subsequent sections document how this condition was identified through sign-in telemetry and Conditional Access evaluation, followed by containment, remediation, and recovery actions.

# Attack Scenario - Conditional Access Exception Abuse

## Scenario Overview

This project simulates how a **legitimate administrative decision made under
business pressure** can unintentionally create a high-risk authentication bypass
for a high-value identity.

No phishing, malware, or external attacker is involved. The risk is created
entirely through internal misconfiguration and exception handling failure.

## Threat Classification

| Attribute | Detail |
|---|---|
| **Attack Type** | Internal CA exception abuse |
| **Attack Vector** | Policy-based MFA exclusion |
| **Root Cause** | Inadequate exception governance |
| **Threat Actor** | Non-malicious internal administrative action |
| **Impact** | Single-factor auth exposure on high-value identity |

## Attack Timeline

**T0 - Secure Baseline**
- CA policy enforces MFA for all users
- CEO authenticates successfully with MFA
- Sign-in logs confirm full enforcement

**T1 - Authentication Disruption**
- CEO replaces mobile device
- Microsoft Authenticator registration becomes unavailable
- MFA challenges cannot be completed
- CEO reports inability to access Microsoft 365 services

**T2 - Emergency Access Request**
- Business impact identified as high priority
- IT approves temporary access workaround
- No formal exception process followed

**T3 - CA Exception Introduced**
- CEO account explicitly excluded from CA MFA policy
- Policy remains enabled for all other users
- No expiration, monitoring, or review configured

**T4 - Security Degradation**
- CA evaluates as Not Applied for CEO sign-ins
- Authentication downgrades to single-factor
- Successful sign-ins without MFA enforcement begin

**T5 - Silent Exposure**
- No automated alerts triggered
- Environment appears compliant at policy level
- High-value account unprotected until manual review

## Why This Represents an Attack Path

Although no malicious activity occurred, the resulting condition is
functionally equivalent to an identity attack surface:

- A high-value account could authenticate without MFA
- Authentication logs showed downgraded security requirements
- No default alerts indicated loss of protection
- Exposure persisted silently

This demonstrates how CA exceptions can override security intent,
creating an attack surface without external compromise.

## Transition

Attack scenario concludes once the CA exclusion is active and the CEO
begins authenticating without MFA enforcement. Detection phase follows.

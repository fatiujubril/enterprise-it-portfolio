# Attack Scenario - Conditional Access Exception Abuse

## Scenario Overview

This project simulates how a legitimate administrative decision made under
business pressure can unintentionally create a high-risk authentication bypass
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

---

## Attack Timeline

```
T0 - SECURE BASELINE
+--------------------------------------------------+
| CA policy enforces MFA for ALL users             |
| CEO authenticates successfully with MFA          |
| Sign-in logs: CA = Success, Auth = MFA           |
+--------------------------------------------------+
                        |
                        v
T1 - AUTHENTICATION DISRUPTION
+--------------------------------------------------+
| CEO replaces mobile device                       |
| Microsoft Authenticator registration lost        |
| MFA challenges cannot be completed               |
| CEO cannot access Microsoft 365 services         |
+--------------------------------------------------+
                        |
                        v
T2 - EMERGENCY ACCESS REQUEST
+--------------------------------------------------+
| CEO reports business-critical access loss        |
| IT team approves temporary workaround            |
| No formal security review conducted              |
| No exception governance process followed         |
+--------------------------------------------------+
                        |
                        v
T3 - CA EXCEPTION INTRODUCED [CONTROL GAP]
+--------------------------------------------------+
| CEO account added to CA policy exclusion list    |
| No expiration date configured                    |
| No monitoring or review scheduled                |
| Policy remains enabled for all other users       |
+--------------------------------------------------+
                        |
                        v
T4 - SILENT MFA BYPASS BEGINS [EXPOSURE]
+--------------------------------------------------+
| CA evaluates as: NOT APPLIED for CEO             |
| Auth requirement downgrades to: SINGLE-FACTOR    |
| CEO authenticates without MFA challenge          |
| Environment appears compliant at policy level    |
| NO automated alert fires                         |
+--------------------------------------------------+
                        |
                        v
T5 - DETECTION VIA MANUAL TELEMETRY REVIEW
+--------------------------------------------------+
| Sign-in log review identifies anomaly            |
| CA = Not Applied on high-value account           |
| Auth = Single-factor on executive identity       |
| Root cause confirmed: explicit policy exclusion  |
| Containment and remediation initiated            |
+--------------------------------------------------+
```

---

## Attack Conditions at T4

The following conditions existed during the exposure window:

| Condition | State |
|---|---|
| CA policy enabled | Yes - for all other users |
| CEO in exclusion list | Yes - explicit exclusion |
| CA evaluation for CEO | Not Applied |
| Auth requirement for CEO | Single-factor |
| Automated alert | None triggered |
| Exposure visibility | Silent - no indication in UI |

At T4 the environment entered a **high-risk identity state** despite
appearing fully secure at a policy configuration level.

---

## Why This Represents an Attack Path

Although no malicious activity occurred, the resulting condition is
functionally equivalent to an identity attack surface:

- A high-value account could authenticate without MFA
- Authentication logs showed downgraded security requirements
- No default alerts indicated loss of protection
- Exposure persisted silently

This demonstrates how CA exceptions can override security intent,
creating an exploitable attack surface without external compromise.

---

## Transition to Detection

Attack scenario concludes at T4 once the CA exclusion is active and the CEO
begins authenticating without MFA enforcement. Detection phase begins at T5.

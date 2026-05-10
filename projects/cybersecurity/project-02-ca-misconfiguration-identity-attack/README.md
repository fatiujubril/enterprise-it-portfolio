# Project 02 - Conditional Access Misconfiguration & Identity Attack Simulation

> **Type:** Identity Security Simulation / Detection Engineering
> **Environment:** Microsoft 365 / Microsoft Entra ID
> **Difficulty:** Intermediate - Advanced
> **Status:** Complete

---

## Executive Summary

A high-value executive account was silently exposed to single-factor authentication
due to a Conditional Access policy exception introduced under business pressure.
No malware, phishing, or external attacker was involved.

This project simulates how a **legitimate administrative decision** creates an
exploitable identity attack path, and demonstrates the full detection and response
lifecycle using sign-in telemetry and Conditional Access policy analysis.

**Key Outcome:** MFA bypass detected via sign-in telemetry. Exception removed.
MFA re-enrolled on new device. Conditional Access enforcement fully restored.
Governance controls recommended to prevent recurrence.

---

## Threat Scenario

| Field | Detail |
|---|---|
| **Incident Type** | Conditional Access Exception Abuse - MFA Bypass |
| **Attack Vector** | Internal policy misconfiguration (no external attacker) |
| **Affected Identity** | CEO - high-value cloud identity |
| **Root Cause** | CA policy exclusion with no expiration or review control |
| **Impact** | Single-factor authentication on executive account |
| **Detection Source** | Entra ID interactive sign-in logs |
| **Resolution** | Exception removed, MFA re-enrolled, enforcement restored |

**No malware. No phishing. No external attacker.**
This is an identity governance failure - one of the most common and under-detected
risk patterns in Microsoft 365 environments.

---

## Environment

| Component | Detail |
|---|---|
| **Identity Platform** | Microsoft Entra ID (cloud-only) |
| **Licensing** | Microsoft 365 Business Standard + Entra ID P2 (trial) |
| **Tenant Type** | Dedicated lab tenant - no pre-existing policies |
| **User Model** | Native cloud users only |
| **MFA Method** | Microsoft Authenticator |
| **Conditional Access** | Single MFA enforcement policy at baseline |

---

## Attack Chain & MITRE ATT&CK Mapping

```
[T1078] Valid Account - Secure baseline, MFA enforced for all users
      |
      v
[T1556.006] Device change disrupts MFA - CEO cannot authenticate
      |
      v
[T1556] CA exception introduced - CEO excluded from MFA policy
      |
      v
[T1078] CEO authenticates with single-factor - CA evaluates Not Applied
      |
      v
[T1556] Exposure persists silently - no automated alert fires
```

| Technique | ID | Description |
|---|---|---|
| Valid Accounts | T1078 | High-value account targeted due to reduced authentication controls |
| Modify Authentication Process | T1556 | CA exception modifies effective authentication requirement |
| Multi-Factor Authentication Request Generation | T1621 | MFA bypassed due to policy exclusion |

---

## Incident Timeline

### T0 - Secure Baseline
MFA enforced for all users via Conditional Access. CEO authenticates successfully with MFA.
No exclusions. No anomalies.

### T1 - Authentication Disruption
CEO replaces mobile device. Microsoft Authenticator registration becomes unavailable.
MFA challenges cannot be completed. Business access disrupted.

### T2 - Emergency Exception Introduced
IT excludes CEO account from MFA Conditional Access policy to restore access.
No expiration, no approval tracking, no follow-up review configured.

### T3 - Silent MFA Bypass
CA evaluates as **Not Applied** for CEO sign-ins.
Authentication downgrades to **single-factor**.
Environment appears compliant at policy level. No alert fires.

### T4 - Detection via Telemetry
Manual review of sign-in logs reveals CA not applied and single-factor auth on CEO account.
Root cause identified: explicit policy exclusion.

### T5 - Containment & Recovery
Exception removed. MFA re-enrolled on new device. CA enforcement restored and validated.

---

## Project Structure

| Folder | Content |
|---|---|
| [00-Executive-Summary](00-Executive-Summary/executive-summary.md) | Full incident overview and key findings |
| [01-Environment-Setup](01-Environment-Setup/tenant-overview.md) | Tenant config, CA baseline, logging architecture |
| [02-Attack-Simulation](02-Attack-Simulation/attack-scenario.md) | Attack timeline, misconfiguration details |
| [03-Detection-Analysis](03-Detection-Analysis/detection-logic.md) | Detection strategy, investigation notes, KQL queries |
| [04-Incident-Response](04-Incident-Response/containment.md) | Containment, remediation, recovery |
| [05-Lessons-Learned](05-Lessons-Learned/root-cause-analysis.md) | Root cause analysis, security improvements |
| [06-GRC-Alignment](06-GRC-Alignment/mitre-attack-mapping.md) | MITRE mapping, compliance framework alignment |
| [Evidence](Evidence/evidence.md) | Full evidence index with screenshot descriptions |

---

## Key Takeaways

- Conditional Access policies can appear secure while silently failing to apply
- MFA configured does not equal MFA enforced
- High-value identities should never rely on ad-hoc policy exceptions
- Sign-in telemetry is the authoritative source for identity security validation
- Governance failures create attack paths without any malicious intent

---

## Skills Demonstrated

Microsoft Entra ID, Conditional Access Policy Design, MFA Enforcement Validation,
Identity Threat Modeling, Sign-in Log Analysis, KQL Detection Queries,
MITRE ATT&CK Mapping, Incident Response Lifecycle, Security Governance,
Exception Handling Controls, Technical Documentation

---

## How to Review This Project

- Start with **00-Executive-Summary** for the full picture in 60 seconds
- Follow the **numbered folders** in order for the full incident lifecycle
- Review **03-Detection-Analysis/kql-queries.md** for detection engineering detail
- Review **06-GRC-Alignment** for MITRE mapping and compliance framework coverage

*All activity performed in a controlled lab tenant. No real user data included.*

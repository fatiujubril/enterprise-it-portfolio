# Project 03 - Privileged Identity & Cloud Security Program

> **Type:** Security Engineering Program / Identity Governance / Cloud Security
> **Environment:** Microsoft Entra ID / Azure / Microsoft Sentinel
> **Difficulty:** Advanced
> **Status:** In Progress

---

## Executive Summary

This project documents the design, implementation, and validation of an
enterprise-grade privileged identity governance and cloud security program
across Microsoft Entra ID and Azure.

Rather than simulating a single incident, this project builds a **complete
security program** â€” covering identity governance, cloud security posture,
detection engineering, GRC, and incident response â€” the way a senior
Identity & Cloud Security Engineer would deliver it in a real organization.

**Program Outcomes:**
- Zero Trust identity controls enforced via Conditional Access and PIM
- Privileged access governed through JIT activation and access reviews
- Azure cloud security posture baselined and hardened via Defender for Cloud
- 8 detection rules built and tuned in Microsoft Sentinel mapped to MITRE ATT&CK
- GRC artifacts produced: risk register, control mapping, exception register
- Tabletop exercise executed: compromised admin + Azure privilege abuse scenario

---

## Threat Context

Identity is the primary attack surface in modern cloud environments.
Microsoft reports that over 90% of cloud breaches involve compromised
or misconfigured identity controls.

This program addresses the most common identity and cloud security failures:
- Permanent privileged role assignments (no JIT, no time-bound access)
- MFA not enforced via Conditional Access for privileged identities
- Cloud security posture not baselined or continuously monitored
- No detection coverage for identity-based attack techniques
- No governance process for access reviews or policy exceptions

---

## Environment

| Component | Detail |
|---|---|
| **Identity Platform** | Microsoft Entra ID (cloud-only) |
| **Cloud Platform** | Microsoft Azure |
| **SIEM** | Microsoft Sentinel |
| **PAM** | Entra ID Privileged Identity Management (PIM) |
| **CSPM** | Microsoft Defender for Cloud |
| **Licensing** | Microsoft 365 + Entra ID P2 + Azure subscription |

---

## Program Structure

| Phase | Folder | Focus |
|---|---|---|
| Architecture | [01-Architecture](01-Architecture/solution-architecture.md) | Zero Trust design, environment overview |
| Identity Governance | [02-Identity-Governance](02-Identity-Governance/pim-design.md) | PIM, CA policies, access reviews, lifecycle |
| Cloud Security Posture | [03-Cloud-Security-Posture](03-Cloud-Security-Posture/defender-for-cloud.md) | CSPM, Azure Policy, benchmark alignment |
| Detection Engineering | [04-Detection-Engineering](04-Detection-Engineering/detection-rules.md) | Sentinel rules, MITRE mapping, tuning |
| GRC | [05-GRC](05-GRC/risk-register.md) | Risk register, control mapping, exceptions |
| Incident Tabletop | [06-Incident-Tabletop](06-Incident-Tabletop/scenario.md) | Compromised admin + Azure abuse scenario |

---

## MITRE ATT&CK Coverage

| # | Detection Rule | Technique ID | Technique |
|---|---|---|---|
| 1 | PIM role activation outside business hours | T1078 | Valid Accounts |
| 2 | Global Admin assigned outside PIM | T1078.004 | Cloud Accounts |
| 3 | MFA disabled for privileged account | T1556 | Modify Authentication Process |
| 4 | Conditional Access policy modified or disabled | T1556 | Modify Authentication Process |
| 5 | Impossible travel on admin account | T1078 | Valid Accounts |
| 6 | Azure Owner/Contributor role assignment | T1098 | Account Manipulation |
| 7 | Defender for Cloud high severity finding | T1190 | Exploit Public-Facing Application |
| 8 | Break-glass account sign-in detected | T1078 | Valid Accounts |

---

## Key Engineering Decisions

- **PIM over permanent roles** - all privileged access is time-bound and approval-gated
- **Conditional Access as enforcement layer** - MFA registration alone is insufficient
- **Sentinel as detection platform** - custom KQL rules over default alert rules
- **Defender for Cloud as CSPM** - continuous posture monitoring over point-in-time audits
- **NIST CSF as GRC framework** - industry-standard control mapping

---

## Skills Demonstrated

Microsoft Entra ID, Privileged Identity Management, Conditional Access,
Zero Trust Architecture, Microsoft Defender for Cloud, Azure Policy,
Microsoft Sentinel, KQL Detection Engineering, MITRE ATT&CK Mapping,
Risk Register, Control Mapping, GRC, Incident Tabletop, Security Program Design

---

## How to Review This Project

- Start with **01-Architecture** for the full environment and design context
- Follow **02-Identity-Governance** for the identity controls implementation
- Review **04-Detection-Engineering/detection-rules.md** for KQL engineering depth
- Review **05-GRC** for business and compliance maturity
- End with **06-Incident-Tabletop** for the operational validation

*All activity performed in a controlled lab environment.*

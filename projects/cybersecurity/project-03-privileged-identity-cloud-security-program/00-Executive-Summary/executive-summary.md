# Executive Summary
## FatiuLab Security — Privileged Identity & Cloud Security Program

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Program Scope:** Identity Governance, Cloud Security Posture, Detection Engineering, GRC  
**Regulatory Context:** PCI-DSS, PIPEDA  
**Program Duration:** May 2026  
**Program Owner:** Global Administrator

---

## 1. Business Context

FatiuLab Security is a 200-person financial technology company processing customer payment data and operating under PCI-DSS and PIPEDA regulatory requirements. As a cloud-first organization built on Microsoft 365 and Azure, identity is the primary attack surface — compromised credentials and misconfigured access controls represent the most significant threat to payment data confidentiality and business continuity.

This program was initiated to address a fundamental security gap: **the absence of a formal, governed, and monitored privileged identity and cloud security program.** Without such a program, the organization faces:

- Permanent standing privilege for all admin accounts — unlimited blast radius on compromise
- No detection capability for identity-based attacks
- No documented compliance posture against PCI-DSS or industry benchmarks
- No tested incident response capability

---

## 2. Program Objectives

| Objective | Target | Achieved |
|---|---|---|
| Eliminate standing privilege | JIT access for all privileged roles | ✅ PIM deployed — no standing privilege |
| Enforce Zero Trust authentication | MFA on every authentication | ✅ 5 CA policies active |
| Establish cloud security baseline | Secure Score > 60% | ✅ 67% achieved |
| Meet CIS benchmark compliance | > 75% controls passing | ✅ 81% (66/81) |
| Deploy identity threat detection | 6+ detection rules | ✅ 8 rules deployed |
| Document and govern all risks | Risk register with residual tracking | ✅ 12 risks documented |
| Validate program effectiveness | Incident tabletop exercise | ✅ Completed — 7-item backlog |

---

## 3. Program Outcomes

### 3.1 Identity Governance

A Zero Trust identity governance framework was implemented across four components:

**Conditional Access:** Five policies enforce MFA for all users and admin roles, block legacy authentication protocols entirely, and apply a separate MFA challenge for Azure Management access. The break-glass account is excluded via a dedicated group monitored monthly.

**Privileged Identity Management:** All privileged roles converted from permanent active to eligible-only. Global Administrator requires approval and MFA for a 2-hour activation window. Security Administrator self-approves with MFA for a 4-hour window. No admin account has standing privilege.

**Access Reviews:** Two automated reviews — quarterly for privileged groups and monthly for the CA exclusion group. Auto-remove on reviewer non-response ensures inaction equals denial, not approval.

**Identity Lifecycle:** Formal joiner/mover/leaver process defined, including staged MFA onboarding and immediate account disable on departure.

### 3.2 Cloud Security Posture

**Secure Score improvement: 50% → 67% (+17%)**

Defender for Cloud CSPM assessed the Azure environment against the Microsoft Cloud Security Benchmark and identified 10 findings. Five were remediated:
- Storage account shared key access disabled
- Public network access disabled
- Security contact email configured
- High severity alert notifications enabled
- Three exemptions created with documented business justification

**CIS Azure Foundations Benchmark v2.0.0: 66/81 controls (81%)**

Identity and Access Management category fully passing — directly validated by the Week 1 identity governance work. Failing categories are documented with accepted risk justifications.

### 3.3 Detection Engineering

Eight custom analytics rules deployed in Microsoft Sentinel covering the identity attack chain:

| Coverage Area | Rules |
|---|---|
| Privileged access abuse | DET-01, DET-02, DET-06 |
| Authentication control weakening | DET-03, DET-04 |
| Credential theft indicators | DET-05, DET-07 |
| Cloud security findings | DET-08 |

Break-glass sign-in detection (DET-07) deployed as NRT — 1-2 minute detection latency for the highest-risk account in the tenant.

### 3.4 GRC Program

- **12 risks** identified and assessed — all residual risks at Medium or Low
- **NIST CSF mapped** — controls aligned across all five functions (Identify, Protect, Detect, Respond, Recover)
- **7 exceptions** documented with compensating controls and quarterly review dates
- **Incident tabletop** completed — AiTM phishing scenario, 4 of 6 attack events detected, 7-item improvement backlog produced

---

## 4. Risk Posture Summary

| Risk Category | Inherent Risk | Residual Risk | Reduction |
|---|---|---|---|
| Compromised Global Admin | High (15) | Medium (8) | 47% |
| Break-Glass Compromise | Medium (10) | Low (5) | 50% |
| PIM Bypass | High (15) | Medium (8) | 47% |
| CA Policy Bypass | High (12) | Low (4) | 67% |
| MFA Bypass | Critical (20) | Medium (6) | 70% |
| Storage Data Exposure | Critical (20) | Low (3) | 85% |
| Regulatory Non-Compliance | High (15) | Low (4) | 73% |

**No residual risks remain at High or Critical.**

---

## 5. Compliance Summary

| Standard | Result |
|---|---|
| CIS Azure Foundations v2.0.0 | 66/81 controls (81%) |
| Microsoft Cloud Security Benchmark | 61/63 controls (97%) |
| Azure CSPM | 1/1 (100%) |
| PCI-DSS key requirements | Req 7, 8, 10, 11, 12 addressed |
| PIPEDA | Principles 4 and 7 addressed |

---

## 6. Key Findings from Tabletop Exercise

The incident tabletop exercise simulating an AiTM phishing attack against a Security Administrator account revealed:

**Strengths:**
- PIM approval gate successfully blocked Global Administrator escalation
- Storage network isolation prevented data exfiltration despite Owner-level Azure access
- Detection rules correlated correctly — four rules fired across the attack chain

**Gaps:**
- 90-minute dwell time before first detection — scheduled rules create latency
- No detection for AiTM session hijack — push notification MFA is susceptible
- No automated containment — manual response adds 12+ minutes to containment time

**Top improvement priorities:**
1. Phishing-resistant MFA (FIDO2) — eliminates AiTM attack vector entirely
2. Sentinel automation playbook — automated account disable on P1 identity incidents
3. Convert critical detection rules to NRT — reduce detection latency from 60 minutes to 2 minutes

---

## 7. Program Value

This program transforms FatiuLab Security's identity and cloud security posture from ad-hoc to governed:

| Before | After |
|---|---|
| Permanent admin privilege | JIT access — 2-4 hour windows only |
| No MFA enforcement | MFA on every authentication |
| Legacy auth enabled | Legacy auth blocked entirely |
| No cloud security baseline | 67% Secure Score, CIS 81% |
| No threat detection | 8 detection rules, 11 data sources |
| No documented risks | 12 risks with residual tracking |
| No tested response | Tabletop completed, backlog created |

The program provides audit-ready evidence across all PCI-DSS and PIPEDA requirements in scope, a defensible compliance posture against the CIS Azure Foundations Benchmark, and a continuously improving detection and response capability.

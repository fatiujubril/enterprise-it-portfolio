# Project 03 — Privileged Identity & Cloud Security Program

**Stack:** Microsoft Entra ID · Azure · Microsoft Sentinel · Defender for Cloud · PIM · Conditional Access · KQL  
**Frameworks:** NIST CSF · CIS Azure Foundations v2.0.0 · MITRE ATT&CK · PCI-DSS · PIPEDA  
**Status:** Complete

---

## Overview

Enterprise-grade Privileged Identity & Cloud Security Program built from scratch on Microsoft Entra ID and Azure — demonstrating the depth, design thinking, and documentation quality expected of an Identity & Cloud Security Engineer.

FatiuLab Security is a fictional 200-person FinTech company processing customer payment data, regulated under PCI-DSS and PIPEDA. Every control decision is grounded in real business risk, and every finding is backed by lab evidence.

This is not a checklist of configurations. It is a complete security program — from Zero Trust identity governance through cloud security posture management, detection engineering, GRC risk management, and a tested incident response capability.

---

## What This Project Proves

**Identity Governance:**
- Zero Trust Conditional Access policy design — 5 policies, MFA enforcement, legacy auth eliminated
- Just-In-Time privileged access via PIM — no standing privilege, approval workflows, full audit trail
- Time-bound eligible assignments — quarterly and semi-annual expiry enforced
- Automated access reviews — auto-remove on non-response, monthly monitoring of CA exclusions

**Cloud Security Posture:**
- Defender for Cloud CSPM — Secure Score improved from 50% to 67% (+17%)
- CIS Azure Foundations Benchmark v2.0.0 — 66/81 controls passing (81%)
- Microsoft Cloud Security Benchmark — 61/63 controls passing (97%)
- Storage account hardening — public access disabled, shared key access disabled
- Documented exception register — 7 exceptions with business justification and compensating controls

**Detection Engineering:**
- 8 custom Sentinel analytics rules mapped to MITRE ATT&CK
- 3 data sources: AuditLogs, SigninLogs, AzureActivity
- 4 MITRE techniques covered across 3 tactics
- NRT rule for break-glass account sign-in — 1-2 minute detection latency
- Full tuning log — false positive analysis and planned improvements per rule

**GRC:**
- Risk register — 12 identity and cloud risks, inherent and residual scoring
- NIST CSF control mapping — every control mapped to framework categories
- Exception register — 7 documented exceptions with compensating controls
- Compliance alignment — PCI-DSS, PIPEDA, CIS, NIST CSF cross-referenced throughout

**Incident Tabletop:**
- Scenario: AiTM phishing → Security Admin compromise → Azure privilege abuse
- 5 exercise injects with response discussion
- Full attack and response timeline
- RACI matrix — roles and responsibilities defined
- 7-item improvement backlog with priorities and timelines

---

## Lab Environment

| Component | Detail |
|---|---|
| Tenant | FatiuLab Security (FatiuLabSecurity.onmicrosoft.com) |
| License | Microsoft 365 Business Premium + Entra ID P2 |
| Users | Global Admin, Security Admin, Standard User, Break Glass |
| Groups | SEC-Global-Admins, SEC-Admins, SEC-Standard-Users, SEC-CA-Exclusions, SEC-PIM-Eligible |
| Azure Subscription | Azure subscription 1 — Canada Central |
| Sentinel Workspace | LAW-FatiuLab-Security |
| Resource Group | RG-FatiuLab-Security |

---

## Program Results

| Metric | Result |
|---|---|
| Conditional Access policies | 5 active (CA01–CA05) |
| PIM roles under JIT governance | 2 (Global Admin, Security Admin) |
| Access reviews | 2 (quarterly + monthly) |
| Defender Secure Score | 50% → **67%** (+17%) |
| CIS benchmark compliance | **66/81 controls (81%)** |
| MCSB compliance | **61/63 controls (97%)** |
| Sentinel data connectors | 11 connected |
| Custom detection rules | 8 (7 Scheduled + 1 NRT) |
| MITRE techniques covered | 4 (T1078, T1098, T1190, T1556) |
| Risks documented | 12 |
| Exceptions documented | 7 |
| Residual risks at High/Critical | 0 |

---

## Project Structure

```
project-03-privileged-identity-cloud-security-program/
│
├── 00-Executive-Summary/
│   └── executive-summary.md
│
├── 01-Architecture/
│   ├── solution-architecture.md
│   ├── zero-trust-design.md
│   └── environment-overview.md
│
├── 02-Identity-Governance/
│   ├── conditional-access-design.md    ← Zero Trust CA policy framework
│   ├── pim-design.md                   ← JIT privileged access program
│   ├── access-reviews.md               ← Identity governance review program
│   └── identity-lifecycle.md           ← Joiner/mover/leaver process
│
├── 03-Cloud-Security-Posture/
│   ├── defender-for-cloud.md           ← CSPM baseline and remediation
│   ├── azure-policy.md                 ← CIS benchmark compliance
│   └── security-benchmark.md           ← Cross-framework posture analysis
│
├── 04-Detection-Engineering/
│   ├── sentinel-architecture.md        ← Workspace and connector design
│   ├── detection-rules.md              ← 8 custom KQL analytics rules
│   ├── mitre-coverage.md               ← ATT&CK coverage matrix
│   └── tuning-log.md                   ← False positive and tuning register
│
├── 05-GRC/
│   ├── risk-register.md                ← 12 identity and cloud risks
│   ├── control-mapping.md              ← NIST CSF framework alignment
│   └── exception-register.md           ← 7 documented exceptions
│
├── 06-Incident-Tabletop/
│   ├── scenario.md                     ← AiTM phishing + Azure abuse
│   ├── timeline.md                     ← Attack and response timeline
│   ├── raci.md                         ← Roles and responsibilities
│   └── lessons-learned.md              ← 7-item improvement backlog
│
└── Evidence/
    ├── Phase-01-Identity/
    │   ├── P3-BL-01 through P3-BL-04   ← Baseline setup
    │   ├── CA-Policies/                 ← 19 CA policy screenshots
    │   ├── PIM/                         ← 12 PIM screenshots
    │   └── Access-Reviews/              ← 4 access review screenshots
    ├── Phase-02-Cloud-Security/
    │   ├── Defender-for-Cloud/          ← 15 CSPM screenshots
    │   └── Azure-Policy/                ← 6 CIS benchmark screenshots
    └── Phase-03-Detections/
        ├── Sentinel/                    ← 4 Sentinel setup screenshots
        └── Analytics-Rules/             ← 8 detection rule screenshots
```

---

## Week-by-Week Build

### Week 1 — Identity Governance
Built the Zero Trust identity foundation from scratch in a fresh Entra ID tenant:
- 4 users and 5 security groups with clear role separation
- 5 Conditional Access policies — MFA enforcement, legacy auth block, Azure Management protection
- PIM configuration — JIT access, approval workflows, time-limited activations
- 2 automated access reviews — quarterly privileged groups, monthly CA exclusions
- Break-glass account with offline credentials and monthly monitoring

### Week 2 — Cloud Security Posture
Linked Azure subscription, deployed Defender for Cloud, and hardened the environment:
- Defender CSPM enabled — Foundational + Defender plans
- Resource deployment — storage account with intentional insecure defaults as baseline
- CSPM remediation — Secure Score 50% → 67% through targeted hardening
- CIS Azure Foundations Benchmark v2.0.0 assigned — 66/81 controls passing
- 3 documented exemptions with business justification
- Microsoft Sentinel workspace deployed with 11 data connectors

### Week 3 — Detection Engineering
Built 8 custom analytics rules in Sentinel covering the identity attack chain:
- Simulation activities to generate real log data across AuditLogs, SigninLogs, AzureActivity
- 8 rules deployed — PIM abuse, CA policy changes, impossible travel, Azure role assignments, break-glass sign-in
- MITRE ATT&CK mapping across 3 tactics and 4 techniques
- Tuning log documenting false positive analysis and planned improvements per rule

### Week 4 — GRC + Tabletop
Built the governance and evidence layer that ties the program together:
- Risk register — 12 risks, all residual risks at Medium or Low
- NIST CSF control mapping — every control mapped across Identify/Protect/Detect/Respond/Recover
- Exception register — 7 exceptions with compensating controls and review dates
- Tabletop exercise — AiTM phishing scenario, 5 injects, full response walkthrough
- 7-item improvement backlog including phishing-resistant MFA and automated containment

---

## Detection Rules Summary

| ID | Rule | Severity | Type | MITRE |
|---|---|---|---|---|
| DET-01 | PIM Activation Outside Business Hours | Medium | Scheduled | T1078.004 |
| DET-02 | Global Admin Assigned Outside PIM | High | Scheduled | T1078.004 |
| DET-03 | MFA Disabled for Privileged Account | High | Scheduled | T1556.006 |
| DET-04 | CA Policy Modified or Disabled | High | Scheduled | T1556.009 |
| DET-05 | Impossible Travel on Admin Account | High | Scheduled | T1078.004 |
| DET-06 | Azure Privileged Role Assignment | High | Scheduled | T1098.003 |
| DET-07 | Break-Glass Account Sign-In | High | **NRT** | T1078.004 |
| DET-08 | Defender for Cloud High Severity Finding | High | Scheduled | T1190 |

---

## Compliance Coverage

| Framework | Result |
|---|---|
| CIS Azure Foundations v2.0.0 | 66/81 controls (81%) |
| Microsoft Cloud Security Benchmark | 61/63 controls (97%) |
| Azure CSPM | 1/1 (100%) |
| PCI-DSS key requirements | Req 7, 8, 10, 11, 12 addressed |
| PIPEDA | Principles 4, 7 addressed |
| NIST CSF | Identify, Protect, Detect, Respond, Recover all mapped |

---

## Key Design Decisions

**Why one permanent Global Administrator?**
PIM requires at least one permanent GA to process activation approvals. Without it, a PIM outage would block all privileged access. This is documented as EXP-01 with compensating controls.

**Why is CA05 in Report-only?**
Standard users haven't completed MFA registration. Enforcing before registration locks users out — an availability incident. Staged rollout is the correct enterprise approach. Documented as EXP-07.

**Why Foundational CSPM instead of all Defender plans?**
CWPP plans charge per deployed resource. With zero servers, databases, or containers, enabling workload protection plans provides no benefit and incurs unnecessary cost. Compensating controls documented for each deferred plan.

**Why NRT for break-glass detection only?**
Break-glass sign-in requires immediate detection — any use is a critical security event. Other rules tolerate 1-hour scheduled latency because their attack scenarios develop over longer timeframes. NRT is used where latency materially affects response capability.

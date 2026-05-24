# Security Benchmark Analysis
## FatiuLab Security — Cloud Security Posture Summary

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Regulatory Context:** PCI-DSS, PIPEDA  
**Tenant:** FatiuLabSecurity.onmicrosoft.com  
**Azure Subscription:** Azure subscription 1  
**Policy Author:** Global Administrator  
**Assessment Period:** May 2026  
**Status:** Active — Phase 2 Complete

---

## 1. Overview

This document provides a consolidated security benchmark analysis for the FatiuLab Security Azure environment. It synthesizes results from three complementary assessment frameworks — Microsoft Defender for Cloud Secure Score, Microsoft Cloud Security Benchmark (MCSB), and CIS Azure Foundations Benchmark v2.0.0 — into a single posture report.

Each framework measures a different dimension of cloud security:

```
Defender for Cloud Secure Score
"How secure is our posture?"
Measures risk reduction across all recommendations
Current: 67% (up from 50% baseline)

Microsoft Cloud Security Benchmark
"Are we following Microsoft's security best practices?"
Maps to Zero Trust and cloud security architecture principles
Current: 61/63 controls (97%)

CIS Azure Foundations Benchmark v2.0.0
"Are we compliant with industry security standards?"
Maps to PCI-DSS, ISO 27001, SOC 2
Current: 66/81 controls (81%)
```

Together these three measurements provide a complete picture of cloud security posture — from operational risk reduction to regulatory compliance readiness.

---

## 2. Executive Summary

| Assessment | Baseline | Current | Change | Target |
|---|---|---|---|---|
| Defender Secure Score | 50% | **67%** | +17% | 75%+ |
| MCSB Controls Passing | N/A | **61/63 (97%)** | — | 63/63 |
| CIS Controls Passing | N/A | **66/81 (81%)** | — | 72/81 (89%)+ |
| Critical Recommendations | 0 | 0 | — | 0 |
| High Recommendations | 0 | 0 | — | 0 |
| Medium Recommendations | 0 | 0 | — | 0 |
| Low Recommendations | 10 | 2 | -8 | 0 |

**Overall posture assessment: GOOD**

The FatiuLab Security Azure environment has achieved a strong baseline security posture for a newly deployed environment. Identity and Access Management controls are fully compliant across both CIS and MCSB frameworks. Remaining gaps are concentrated in three areas: storage account advanced encryption, Microsoft Defender workload protection plans (intentionally deferred), and logging/monitoring configuration (planned for Week 3).

---

## 3. Secure Score Analysis

### 3.1 Score Progression

| Phase | Score | Key Action |
|---|---|---|
| Baseline (fresh subscription) | 50% | No remediations applied |
| After Week 2 remediations | **67%** | Storage hardening + security contact + exemptions |
| Target (after Week 3) | 75%+ | Diagnostic settings + activity log alerts |

Evidence:
- [P3-DFC-08-Secure-Score-Baseline.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-08-Secure-Score-Baseline.png) — baseline 50%
- [P3-DFC-15-Secure-Score-After.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-15-Secure-Score-After.png) — current 67%

### 3.2 Remediations Completed

| Remediation | Finding Resolved | Score Impact |
|---|---|---|
| Disabled shared key access | Storage accounts should prevent shared key access | ✅ |
| Disabled public network access | Storage accounts should restrict network access | ✅ |
| Configured security contact email | Subscriptions should have a contact email | ✅ |
| Enabled high severity alert notifications | Email notification for high severity alerts | ✅ |
| Created 3 documented exemptions | Permanent GA, Defender RM, Defender Storage | ✅ |

### 3.3 Remaining Findings

| Finding | Severity | Resource | Decision |
|---|---|---|---|
| Storage account should use a private link connection | Low | fatiulabstorage01 | Accepted risk |
| There should be more than one owner assigned | Low | Azure subscription 1 | Accepted risk |

---

## 4. Microsoft Cloud Security Benchmark Analysis

### 4.1 Results Summary

**61 of 63 controls passing (97%)**

Evidence: [P3-AZP-03-CIS-Compliance-Dashboard.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-03-CIS-Compliance-Dashboard.png)

### 4.2 Control Category Results

| Category | Status | Detail |
|---|---|---|
| NS. Network Security | ❌ | Private link connection finding — accepted risk |
| IM. Identity Management | ✅ | Full compliance — Week 1 identity governance |
| PA. Privileged Access | ❌ | Permanent GA exempted — intentional design |
| DP. Data Protection | ✅ | Storage encryption and transfer controls passing |
| AM. Asset Management | ✅ | Resource inventory and tagging compliant |
| LT. Logging and Threat Detection | ✅ | Sentinel workspace connected |
| IR. Incident Response | ✅ | Security contact and alert notifications configured |
| PV. Posture and Vulnerability Management | ✅ | CSPM enabled, recommendations tracked |
| ES. Endpoint Security | ✅ | No endpoints deployed |
| BR. Backup and Recovery | ✅ | No applicable resources |
| DS. DevOps Security | ✅ | No applicable resources |
| GS. Governance and Strategy | ⚪ | Not yet evaluated |

### 4.3 Key Observations

**Identity Management fully passing** is the most significant result. The Week 1 identity governance program — Conditional Access policies, PIM JIT access, Access Reviews — directly satisfies the MCSB Identity Management category. This demonstrates that the identity controls were not just technically configured correctly but are architecturally aligned with Microsoft's security best practices framework.

**Privileged Access failing** is an intentional, documented exception. The permanent Global Administrator assignment triggers this finding. As documented in `pim-design.md`, one permanent GA is required as the PIM safety anchor and tenant recovery mechanism. The exemption `EXP-01-PIM-Permanent-GA-Intentional` documents this decision with business justification.

---

## 5. CIS Azure Foundations Benchmark Analysis

### 5.1 Results Summary

**66 of 81 controls passing (81%)**

Evidence:
- [P3-AZP-05-CIS-Benchmark-Results.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-05-CIS-Benchmark-Results.png) — summary
- [P3-AZP-05b-CIS-Benchmark-Controls-Detail.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-05b-CIS-Benchmark-Controls-Detail.png) — control detail

### 5.2 Category Results

| CIS Category | Status | Controls |
|---|---|---|
| 1. Identity and Access Management | ✅ | All passing |
| 2. Microsoft Defender | ❌ | 3 failing — intentionally disabled |
| 3. Storage Accounts | ❌ | 3 failing — accepted risk |
| 4. Database Services | ✅ | All passing (no databases) |
| 5. Logging and Monitoring | ❌ | 2 failing — planned Week 3 |
| 6. Networking | ✅ | All passing |
| 7. Virtual Machines | ✅ | All passing (no VMs) |
| 8. Key Vault | ✅ | All passing (no Key Vaults) |
| 9. AppService | ✅ | All passing (no App Services) |

### 5.3 Failing Control Analysis

**8 failing controls across 3 categories — breakdown by decision type:**

| Decision Type | Count | Controls |
|---|---|---|
| Accepted risk — cost avoided | 3 | Defender for Databases, Cosmos DB, Resource Manager |
| Accepted risk — lab scope | 3 | Infrastructure encryption, Private endpoint, CMK |
| Planned remediation — Week 3 | 2 | Diagnostic settings, Activity log alerts |

**Zero unmanaged gaps** — every failing control has a documented decision and owner.

---

## 6. Cross-Framework Mapping

The three frameworks assess overlapping but distinct control areas. This table shows how the same underlying control appears across frameworks:

| Security Control | Defender Score | MCSB | CIS v2.0.0 |
|---|---|---|---|
| MFA for all users | ✅ | IM. Identity Management ✅ | 1. IAM ✅ |
| Privileged access (PIM) | ⚠️ Exempted | PA. Privileged Access ❌ | 1. IAM ✅ |
| Legacy auth blocked | ✅ | IM. Identity Management ✅ | 1. IAM ✅ |
| Storage public access | ✅ | NS. Network Security ❌ | 3. Storage ✅ |
| Storage shared key | ✅ | DP. Data Protection ✅ | 3. Storage ✅ |
| Security contact | ✅ | IR. Incident Response ✅ | 2.1.19 ✅ |
| High severity alerts | ✅ | IR. Incident Response ✅ | 2.1.20 ✅ |
| Diagnostic settings | ❌ | LT. Logging ✅ | 5. Logging ❌ |
| Private endpoint | ❌ | NS. Network Security ❌ | 3.10 ❌ |
| Defender CWPP plans | ⚠️ Exempted | PV. Posture ✅ | 2. Defender ❌ |

**Key insight:** The same control can appear as passing in one framework and failing in another depending on how strictly each framework defines compliance. PIM's permanent GA exemption passes CIS Category 1 (IAM) because CIS doesn't specifically require zero permanent GA assignments — but fails MCSB Privileged Access because Microsoft's benchmark explicitly recommends eliminating all permanent active assignments. Understanding these nuances is what distinguishes a security engineer from someone who just reads dashboard numbers.

---

## 7. Posture Improvement Roadmap

### 7.1 Week 3 — Logging and Monitoring (Highest Priority)

Resolving the CIS Category 5 findings in Week 3 will:
- Resolve 2 failing CIS controls (5.1 and 5.2)
- Improve CIS score from 66/81 to approximately 68/81
- Feed activity log alerts into Sentinel detection rules
- Close the most actionable gap in the current posture

**Actions:**
- Deploy diagnostic settings on fatiulabstorage01
- Configure activity log alerts for critical operations
- Connect to Sentinel analytics rules

### 7.2 Accepted Risk Register

The following gaps are formally accepted and will not be remediated:

| Control | Framework | Justification | Reviewed By | Date |
|---|---|---|---|---|
| Private endpoint for storage | CIS 3.10, MCSB NS | Lab environment — private endpoint deferred | Global Admin | May 2026 |
| Infrastructure encryption | CIS 3.2 | Lab environment — MMK sufficient | Global Admin | May 2026 |
| Customer managed keys | CIS 3.12 | Lab environment — cost and complexity avoided | Global Admin | May 2026 |
| Defender CWPP plans | CIS 2, MCSB | No applicable resources — cost avoided, exempted | Global Admin | May 2026 |
| Permanent GA assignment | MCSB PA | PIM safety anchor — intentional design | Global Admin | May 2026 |

### 7.3 Target State Metrics

| Metric | Current | After Week 3 | After Full Maturity |
|---|---|---|---|
| Secure Score | 67% | 75%+ | 85%+ |
| CIS Controls | 66/81 (81%) | 68/81 (84%) | 72/81 (89%)+ |
| MCSB Controls | 61/63 (97%) | 62/63 (98%) | 63/63 (100%) |
| Unmanaged gaps | 0 | 0 | 0 |

---

## 8. Compliance Alignment Summary

| Control Area | PCI-DSS | PIPEDA | CIS v2.0.0 | NIST CSF | Status |
|---|---|---|---|---|---|
| Identity governance | Req 7, 8 | Principle 4, 7 | Category 1 | PR.AC | ✅ Complete |
| Network security | Req 1 | Principle 7 | Category 6 | PR.AC-3 | ✅ Passing |
| Data protection | Req 3, 4 | Principle 7 | Category 3 | PR.DS | ⚠️ Partial |
| Logging and monitoring | Req 10 | Principle 4 | Category 5 | DE.CM | 🔨 Week 3 |
| Incident response | Req 12 | Principle 7 | Category 2 | RS | ✅ Passing |
| Vulnerability management | Req 6, 11 | Principle 7 | Category 2 | ID.RA | ⚠️ CWPP deferred |

---

## 9. Evidence Index

### Defender for Cloud
| File | Description |
|---|---|
| [P3-DFC-08-Secure-Score-Baseline.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-08-Secure-Score-Baseline.png) | Secure Score baseline — 50% |
| [P3-DFC-09-Recommendations.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-09-Recommendations.png) | Baseline recommendations — 10 Low |
| [P3-DFC-10-Remediation-SharedKey-Disabled.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-10-Remediation-SharedKey-Disabled.png) | Shared key access disabled |
| [P3-DFC-11-Remediation-NetworkAccess-Disabled.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-11-Remediation-NetworkAccess-Disabled.png) | Public network access disabled |
| [P3-DFC-12-Remediation-SecurityContact.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-12-Remediation-SecurityContact.png) | Security contact configured |
| [P3-DFC-13-Exemptions.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-13-Exemptions.png) | Three exemptions with justification |
| [P3-DFC-14-Recommendations-After.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-14-Recommendations-After.png) | Recommendations after — 2 Low remaining |
| [P3-DFC-15-Secure-Score-After.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-15-Secure-Score-After.png) | Secure Score after — 67% |

### Azure Policy
| File | Description |
|---|---|
| [P3-AZP-01-CIS-Policy-Assignment.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-01-CIS-Policy-Assignment.png) | CIS initiative assignment |
| [P3-AZP-02-CIS-Policy-Created.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-02-CIS-Policy-Created.png) | Policy assignments list |
| [P3-AZP-03-CIS-Compliance-Dashboard.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-03-CIS-Compliance-Dashboard.png) | MCSB regulatory compliance dashboard |
| [P3-AZP-04-CIS-Compliance-CSPM-Tab.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-04-CIS-Compliance-CSPM-Tab.png) | Azure CSPM 1/1 passing |
| [P3-AZP-05-CIS-Benchmark-Results.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-05-CIS-Benchmark-Results.png) | CIS v2.0.0 — 66/81 controls |
| [P3-AZP-05b-CIS-Benchmark-Controls-Detail.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-05b-CIS-Benchmark-Controls-Detail.png) | CIS individual control detail |

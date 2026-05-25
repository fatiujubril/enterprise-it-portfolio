# Evidence Index
## FatiuLab Security — Program Evidence Register

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Last Updated:** May 2026

---

## 1. Overview

This index maps every piece of evidence collected during the Privileged Identity & Cloud Security Program to the control or finding it supports. All evidence files are screenshots taken directly from the lab environment during implementation.

**Evidence naming convention:** `P3-[PHASE]-[SEQUENCE]-[Description].png`

---

## 2. Phase 01 — Identity Governance

### 2.1 Baseline Setup

| File | Description | Controls Evidenced |
|---|---|---|
| [P3-BL-01-Tenant-Overview.png](../Evidence/Phase-01-Identity/P3-BL-01-Tenant-Overview.png) | FatiuLab Security tenant overview — Entra ID P2 licensed | ID.AM-1, environment baseline |
| [P3-BL-02-P2-License-Assigned.png](../Evidence/Phase-01-Identity/P3-BL-02-P2-License-Assigned.png) | M365 Business Premium + Entra ID P2 assigned | PR.AC-6, licensing baseline |
| [P3-BL-03-Users-List.png](../Evidence/Phase-01-Identity/P3-BL-03-Users-List.png) | All 4 users — GA, Security Admin, Standard User, Break Glass | ID.AM-6, PR.AC-1 |
| [P3-BL-04-Security-Groups.png](../Evidence/Phase-01-Identity/P3-BL-04-Security-Groups.png) | All 5 security groups created | PR.AC-4, group-based access control |

### 2.2 Conditional Access Policies

| File | Description | Controls Evidenced |
|---|---|---|
| [P3-CA-00-Policy-List.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-00-Policy-List.png) | All 5 CA policies — names, states, overview | PR.AC-3, PR.AC-7, Zero Trust |
| [P3-CA-01-Require-MFA-All-Users.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-01-Require-MFA-All-Users.png) | CA01 — MFA all users, grant control | PR.AC-7, PCI-DSS Req 8.4 |
| [P3-CA-01b-Require-MFA-All-Users-Include.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-01b-Require-MFA-All-Users-Include.png) | CA01 — all users include scope | PR.AC-7 |
| [P3-CA-01c-Require-MFA-All-Users-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-01c-Require-MFA-All-Users-Exclude.png) | CA01 — SEC-CA-Exclusions exclude scope | EXP-07 compensating control |
| [P3-CA-02-Require-MFA-Admins.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02-Require-MFA-Admins.png) | CA02 — MFA admins, grant control | PR.AC-4, PCI-DSS Req 7.2 |
| [P3-CA-02b-Require-MFA-Admins-Include-01.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02b-Require-MFA-Admins-Include-01.png) | CA02 — admin directory roles (scroll 1) | PR.AC-4 |
| [P3-CA-02b-Require-MFA-Admins-Include-02.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02b-Require-MFA-Admins-Include-02.png) | CA02 — admin directory roles (scroll 2) | PR.AC-4 |
| [P3-CA-02c-Require-MFA-Admins-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-02c-Require-MFA-Admins-Exclude.png) | CA02 — exclusion group | EXP-01 compensating control |
| [P3-CA-03-Block-Legacy-Auth.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-03-Block-Legacy-Auth.png) | CA03 — block legacy auth, grant control | PR.PT-3, PCI-DSS Req 6.2 |
| [P3-CA-03b-Block-Legacy-Auth-Conditions.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-03b-Block-Legacy-Auth-Conditions.png) | CA03 — client apps conditions | PR.PT-3 |
| [P3-CA-03c-Block-Legacy-Auth-Users.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-03c-Block-Legacy-Auth-Users.png) | CA03 — all users scope | PR.PT-3 |
| [P3-CA-04-Require-MFA-Azure-Management.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04-Require-MFA-Azure-Management.png) | CA04 — MFA Azure Management | PR.AC-4, RISK-06 |
| [P3-CA-04b-Require-MFA-Azure-Management-Include.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04b-Require-MFA-Azure-Management-Include.png) | CA04 — include scope | PR.AC-4 |
| [P3-CA-04c-Require-MFA-Azure-Management-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04c-Require-MFA-Azure-Management-Exclude.png) | CA04 — exclusion group | EXP-01 |
| [P3-CA-04d-Require-MFA-Azure-Management-App.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-04d-Require-MFA-Azure-Management-App.png) | CA04 — Azure Resource Manager app selected | PR.AC-4 |
| [P3-CA-05-Restrict-Standard-Users.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-05-Restrict-Standard-Users.png) | CA05 — report-only policy | EXP-07 |
| [P3-CA-05b-Restrict-Standard-Users-Include.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-05b-Restrict-Standard-Users-Include.png) | CA05 — SEC-Standard-Users scope | EXP-07 |
| [P3-CA-05c-Restrict-Standard-Users-Exclude.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-05c-Restrict-Standard-Users-Exclude.png) | CA05 — exclusion group | EXP-07 |
| [P3-CA-07-Named-Location.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-07-Named-Location.png) | LOC-FatiuLab-HQ trusted location | Zero Trust network boundary |

### 2.3 Privileged Identity Management

| File | Description | Controls Evidenced |
|---|---|---|
| [P3-PIM-00-Quick-Start.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-00-Quick-Start.png) | PIM enabled and ready | PR.AC-4 baseline |
| [P3-PIM-01-GlobalAdmin-Role-Settings.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-01-GlobalAdmin-Role-Settings.png) | Global Admin — 2hr, MFA, approval required | PR.AC-4, PCI-DSS Req 7.2, RISK-01 |
| [P3-PIM-02-SecurityAdmin-Role-Settings.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-02-SecurityAdmin-Role-Settings.png) | Security Admin — 4hr, MFA, self-approve | PR.AC-4, RISK-03 |
| [P3-PIM-03-Eligible-Assignments.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03-Eligible-Assignments.png) | Both eligible assignments with expiry dates | PR.AC-4 |
| [P3-PIM-03b-Eligible-Assignment-Settings.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03b-Eligible-Assignment-Settings.png) | Eligible assignment creation — 3 month expiry | PR.AC-4 |
| [P3-PIM-03c-Active-Assignment-Enforcement.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03c-Active-Assignment-Enforcement.png) | Active assignment MFA + justification required | EXP-01 compensating control |
| [P3-PIM-03d-Active-Assignments.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03d-Active-Assignments.png) | Active assignments — Global Admin permanent only | EXP-01 |
| [P3-PIM-04-JIT-My-Roles-Eligible.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-04-JIT-My-Roles-Eligible.png) | Security Admin eligible roles before activation | PR.AC-4 JIT workflow |
| [P3-PIM-05-JIT-Activation-Request.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-05-JIT-Activation-Request.png) | JIT activation request with justification | PR.AC-4, PCI-DSS Req 10.2 |
| [P3-PIM-06-JIT-Activation-Confirmed.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-06-JIT-Activation-Confirmed.png) | Role activated — 4 hour window confirmed | PR.AC-4 |
| [P3-PIM-07-Audit-Log.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-07-Audit-Log.png) | PIM admin audit history | DE.CM-3, PCI-DSS Req 10.2 |
| [P3-PIM-07b-Activation-Audit-Log.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-07b-Activation-Audit-Log.png) | Entra ID audit log — JIT activation with justification | DE.CM-3, PCI-DSS Req 10.3 |

### 2.4 Access Reviews

| File | Description | Controls Evidenced |
|---|---|---|
| [P3-AR-00-Access-Reviews-List.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-00-Access-Reviews-List.png) | Both access reviews active | PR.AC-1, PCI-DSS Req 7.2.4 |
| [P3-AR-01-Privileged-Groups-Review-Settings.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-01-Privileged-Groups-Review-Settings.png) | AR-01 quarterly review configuration | PR.AC-1, RISK-08 |
| [P3-AR-01b-Privileged-Groups-Review-Created.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-01b-Privileged-Groups-Review-Created.png) | AR-01 created and active | PR.AC-1 |
| [P3-AR-02-CA-Exclusions-Review-Settings.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-02-CA-Exclusions-Review-Settings.png) | AR-02 monthly CA exclusions review | RISK-02, EXP-01 compensating control |

---

## 3. Phase 02 — Cloud Security Posture

### 3.1 Defender for Cloud

| File | Description | Controls Evidenced |
|---|---|---|
| [P3-DFC-01-Defender-Plans-Before.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-01-Defender-Plans-Before.png) | Baseline — all plans off except Foundational CSPM | Baseline state |
| [P3-DFC-02-Defender-Plans-Enabled.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-02-Defender-Plans-Enabled.png) | Defender CSPM enabled | DE.CM-6, ID.RA-1 |
| [P3-DFC-03-Resource-Group.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-03-Resource-Group.png) | RG-FatiuLab-Security created | Environment setup |
| [P3-DFC-04-Storage-Account-Config.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-04-Storage-Account-Config.png) | Storage account insecure defaults — before state | RISK-07 baseline |
| [P3-DFC-05-Storage-Account-Deployed.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-05-Storage-Account-Deployed.png) | Storage account deployment confirmed | Resource deployment |
| [P3-DFC-06-Storage-Account-Overview.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-06-Storage-Account-Overview.png) | Storage account properties | PR.DS-1 |
| [P3-DFC-07-Inventory.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-07-Inventory.png) | Inventory — 3 resources, 3 unhealthy | ID.AM-1, ID.AM-2 |
| [P3-DFC-08-Secure-Score-Baseline.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-08-Secure-Score-Baseline.png) | Secure Score baseline — 50% | DE.AE-1, RISK-07 before |
| [P3-DFC-09-Recommendations.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-09-Recommendations.png) | Baseline recommendations — 10 Low findings | ID.RA-1 |
| [P3-DFC-10-Remediation-SharedKey-Disabled.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-10-Remediation-SharedKey-Disabled.png) | Shared key access disabled | PR.DS-5, CIS 3.1 |
| [P3-DFC-11-Remediation-NetworkAccess-Disabled.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-11-Remediation-NetworkAccess-Disabled.png) | Public network access disabled | PR.AC-5, CIS 3.8 |
| [P3-DFC-12-Remediation-SecurityContact.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-12-Remediation-SecurityContact.png) | Security contact email configured | RS.CO-2, CIS 2.1.19 |
| [P3-DFC-13-Exemptions.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-13-Exemptions.png) | Three exemptions — EXP-01, EXP-02, EXP-03 | Exception register |
| [P3-DFC-14-Recommendations-After.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-14-Recommendations-After.png) | Recommendations after — 2 Low remaining | Remediation outcome |
| [P3-DFC-15-Secure-Score-After.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-15-Secure-Score-After.png) | Secure Score after — 67% (+17%) | DE.AE-1, RISK-07 after |

### 3.2 Azure Policy

| File | Description | Controls Evidenced |
|---|---|---|
| [P3-AZP-00-Subscription-Linked.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-00-Subscription-Linked.png) | Azure subscription linked to FatiuLab tenant | Environment setup |
| [P3-AZP-01-CIS-Policy-Assignment.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-01-CIS-Policy-Assignment.png) | CIS initiative assignment review | ID.GV-1 |
| [P3-AZP-02-CIS-Policy-Created.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-02-CIS-Policy-Created.png) | Policy assignments list — 3 assignments | ID.GV-1 |
| [P3-AZP-03-CIS-Compliance-Dashboard.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-03-CIS-Compliance-Dashboard.png) | MCSB regulatory compliance — 61/63 categories | PR.IP-1, RISK-10 |
| [P3-AZP-04-CIS-Compliance-CSPM-Tab.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-04-CIS-Compliance-CSPM-Tab.png) | Azure CSPM 1/1 — 100% | DE.CM-6 |
| [P3-AZP-05-CIS-Benchmark-Results.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-05-CIS-Benchmark-Results.png) | CIS v2.0.0 — 66/81 controls (81%) | ID.GV-1, RISK-10 |
| [P3-AZP-05b-CIS-Benchmark-Controls-Detail.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-05b-CIS-Benchmark-Controls-Detail.png) | CIS individual control pass/fail detail | ID.GV-1 |

---

## 4. Phase 03 — Detection Engineering

### 4.1 Sentinel Setup

| File | Description | Controls Evidenced |
|---|---|---|
| [P3-SEN-01-LogAnalytics-Workspace.png](../Evidence/Phase-03-Detections/Sentinel/P3-SEN-01-LogAnalytics-Workspace.png) | LAW-FatiuLab-Security workspace created | PR.PT-1 |
| [P3-SEN-02-Sentinel-Workspace.png](../Evidence/Phase-03-Detections/Sentinel/P3-SEN-02-Sentinel-Workspace.png) | Sentinel enabled on workspace | DE.CM-1 |
| [P3-SEN-03-Data-Connectors.png](../Evidence/Phase-03-Detections/Sentinel/P3-SEN-03-Data-Connectors.png) | 11 connectors connected | DE.AE-3, PR.PT-1 |

### 4.2 Analytics Rules

| File | Description | Controls Evidenced |
|---|---|---|
| [P3-DET-01-PIM-Outside-Hours.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-01-PIM-Outside-Hours.png) | DET-01 — PIM outside hours (T1078.004) | DE.CM-3, RISK-01 |
| [P3-DET-02-GlobalAdmin-Outside-PIM.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-02-GlobalAdmin-Outside-PIM.png) | DET-02 — GA outside PIM (T1078.004) | DE.CM-3, RISK-03 |
| [P3-DET-03-MFA-Disabled.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-03-MFA-Disabled.png) | DET-03 — MFA disabled (T1556.006) | DE.CM-3, RISK-05 |
| [P3-DET-04-CA-Policy-Modified.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-04-CA-Policy-Modified.png) | DET-04 — CA policy modified (T1556.009) | DE.CM-3, RISK-04 |
| [P3-DET-05-Impossible-Travel.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-05-Impossible-Travel.png) | DET-05 — Impossible travel (T1078.004) | DE.CM-7, RISK-01 |
| [P3-DET-06-Azure-Role-Assignment.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-06-Azure-Role-Assignment.png) | DET-06 — Azure role assignment (T1098.003) | DE.CM-1, RISK-06 |
| [P3-DET-07-Break-Glass-Sign-In.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-07-Break-Glass-Sign-In.png) | DET-07 — Break-glass NRT (T1078.004) | DE.CM-7, RISK-02 |
| [P3-DET-08-Defender-High-Severity.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-08-Defender-High-Severity.png) | DET-08 — Defender finding (T1190) | DE.AE-2, RISK-12 |

---

## 5. Evidence Summary

| Phase | Screenshots | Controls Covered |
|---|---|---|
| Phase 01 — Identity | 37 | PR.AC, DE.CM, PR.PT, PCI-DSS Req 7/8/10 |
| Phase 02 — Cloud Security | 22 | ID.AM, ID.GV, ID.RA, PR.DS, PR.IP, RS.CO |
| Phase 03 — Detections | 11 | DE.AE, DE.CM, RS.AN |
| **Total** | **70** | **All NIST CSF functions covered** |

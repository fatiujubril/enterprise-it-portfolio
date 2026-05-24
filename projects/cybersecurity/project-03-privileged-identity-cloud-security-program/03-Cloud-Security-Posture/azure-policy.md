# Azure Policy — Compliance Framework
## FatiuLab Security — CIS Benchmark Enforcement & Regulatory Compliance

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Regulatory Context:** PCI-DSS, PIPEDA  
**Tenant:** FatiuLabSecurity.onmicrosoft.com  
**Azure Subscription:** Azure subscription 1  
**Policy Author:** Global Administrator  
**Last Reviewed:** May 2026  
**Status:** Active — Phase 2 Complete

---

## 1. Overview

Azure Policy is the compliance enforcement and continuous assessment engine for the FatiuLab Security Azure environment. Where Defender for Cloud's Secure Score measures security posture, Azure Policy measures regulatory compliance — evaluating every resource against a defined standard and producing audit-ready evidence of compliance status.

The distinction is important:

```
Defender for Cloud Secure Score    Azure Policy Compliance
(Security Posture)                 (Regulatory Compliance)

"How secure is our environment?"   "Are we compliant with CIS/NIST?"
Measures risk reduction            Measures control adherence
Internal engineering metric        Audit evidence for regulators
Score: 67%                         CIS v2.0.0: 66/81 controls (81%)
```

For a FinTech processing payment data, regulatory compliance evidence is not optional — PCI-DSS requires documented proof that security controls are in place and continuously monitored. Azure Policy automates the evidence collection that would otherwise require manual audits.

---

## 2. Policy Framework Selection

### 2.1 Why CIS Azure Foundations Benchmark

The CIS (Center for Internet Security) Microsoft Azure Foundations Benchmark v2.0.0 was selected as the primary compliance framework for the following reasons:

| Reason | Detail |
|---|---|
| Industry recognition | CIS benchmarks are recognized by PCI-DSS, HIPAA, SOC 2, and ISO 27001 auditors |
| Azure-specific | Written specifically for Microsoft Azure — every control maps to a concrete Azure configuration |
| PCI-DSS alignment | CIS Azure Benchmark controls map directly to PCI-DSS Requirements 6, 7, 8, and 10 |
| Free and built-in | Available as a built-in Azure Policy initiative — no additional licensing required |
| Prescriptive | Unlike NIST which defines outcomes, CIS defines exactly what to configure |

### 2.2 CIS vs NIST — Framework Roles in This Project

| Framework | Role | Where Used |
|---|---|---|
| CIS Azure Foundations v2.0.0 | Technical implementation standard | Azure Policy initiative — enforces specific configurations |
| NIST CSF | Program design framework | Documentation — maps controls to security program categories |
| PCI-DSS | Regulatory requirement | Compliance driver — the "why" behind every control |

CIS tells us **how** to configure Azure securely. NIST tells us **what categories** of controls we need. PCI-DSS tells us **why** we need them. All three work together.

---

## 3. Policy Assignments

### 3.1 Assignment Inventory

| Assignment Name | Type | Scope | Assigned By | Purpose |
|---|---|---|---|---|
| CIS Microsoft Azure Foundations Benchmark v2.0.0 | Initiative | Azure subscription 1 | Global Admin | CIS compliance assessment |
| ASC Default | Initiative | Azure subscription 1 | Azure (automatic) | Microsoft Cloud Security Benchmark |
| Configure Azure Activity logs to stream to Log Analytics | Policy | Azure subscription 1 | Global Admin | Sentinel log ingestion |

Evidence:
- [P3-AZP-01-CIS-Policy-Assignment.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-01-CIS-Policy-Assignment.png) — CIS assignment review page
- [P3-AZP-02-CIS-Policy-Created.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-02-CIS-Policy-Created.png) — assignments list showing all 3 assignments

### 3.2 CIS Initiative — Assignment Details

| Setting | Value |
|---|---|
| Initiative | CIS Microsoft Azure Foundations Benchmark v2.0.0 |
| Version | 1.\*.\* |
| Scope | Azure subscription 1 |
| Exclusions | None |
| Policy enforcement | Default (Audit mode) |
| Assigned by | Global Admin |
| Assignment date | May 2026 |

**Audit mode rationale:**

The CIS initiative is assigned in Audit mode, not Enforce mode. In Audit mode, non-compliant resources are flagged and reported but not automatically blocked or modified. This is the correct approach for an initial compliance assessment — Enforce mode would prevent resource deployments that don't meet CIS requirements, which could disrupt operations during the initial hardening phase.

Enforce mode is the target state after the initial remediation cycle is complete.

---

## 4. CIS Compliance Assessment Results

### 4.1 Overall Compliance

| Standard | Controls Passing | Total Controls | Pass Rate |
|---|---|---|---|
| CIS Azure Foundations v2.0.0 | **66** | **81** | **81%** |
| Microsoft Cloud Security Benchmark | 61 | 63 | 97% |
| Azure CSPM | 1 | 1 | 100% |

Evidence: [P3-AZP-05-CIS-Benchmark-Results.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-05-CIS-Benchmark-Results.png)

### 4.2 CIS Control Category Breakdown

| CIS Category | Status | Notes |
|---|---|---|
| 1. Identity and Access Management | ✅ Passing | PIM + CA policies + MFA enforcement — Week 1 work |
| 2. Microsoft Defender | ❌ Failing | CWPP plans intentionally disabled — cost avoided, exempted |
| 3. Storage Accounts | ❌ Failing | Infrastructure encryption + private endpoints — accepted risk |
| 4. Database Services | ✅ Passing | No databases deployed — no applicable findings |
| 5. Logging and Monitoring | ❌ Failing | Diagnostic settings not yet deployed — Week 3 remediation |
| 6. Networking | ✅ Passing | Public network access disabled on storage account |
| 7. Virtual Machines | ✅ Passing | No VMs deployed — no applicable findings |
| 8. Key Vault | ✅ Passing | No Key Vaults deployed — no applicable findings |
| 9. AppService | ✅ Passing | No App Services deployed — no applicable findings |

Evidence:
- [P3-AZP-05-CIS-Benchmark-Results.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-05-CIS-Benchmark-Results.png) — category summary
- [P3-AZP-05b-CIS-Benchmark-Controls-Detail.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-05b-CIS-Benchmark-Controls-Detail.png) — individual control pass/fail detail

### 4.2b CIS Control Detail — Failing Categories

**Category 1 — Identity and Access Management: PASSING ✅**

| Control | Status |
|---|---|
| 1.5 Ensure Guest Users Are Reviewed on a Regular Basis | ✅ |
| 1.23 Ensure That No Custom Subscription Administrator Roles Exist | ✅ |

**Category 2 — Microsoft Defender: FAILING ❌**

| Control | Status | Reason |
|---|---|---|
| 2.1.1 Defender for Servers | ✅ | Not applicable — no servers |
| 2.1.2 Defender for App Services | ✅ | Not applicable |
| 2.1.3 Defender for Databases | ❌ | Intentionally disabled — no databases deployed |
| 2.1.4 Defender for Azure SQL | ✅ | Not applicable |
| 2.1.5 Defender for SQL on Machines | ✅ | Not applicable |
| 2.1.6 Defender for Open-Source Databases | ✅ | Not applicable |
| 2.1.7 Defender for Storage | ⚪ | Not evaluated |
| 2.1.8 Defender for Containers | ✅ | Not applicable |
| 2.1.9 Defender for Cosmos DB | ❌ | Intentionally disabled |
| 2.1.10 Defender for Key Vault | ✅ | Not applicable |
| 2.1.12 Defender for Resource Manager | ❌ | Intentionally disabled — exempted EXP-02 |
| 2.1.13 Apply System Updates | ✅ | |
| 2.1.17 Auto-provisioning for Containers | ✅ | |
| 2.1.19 Security Contact Email Configured | ✅ | Remediated — security contact configured |
| 2.1.20 Notify about High Severity Alerts | ✅ | Remediated |

**Category 3 — Storage Accounts: FAILING ❌**

| Control | Status | Reason |
|---|---|---|
| 3.1 Secure Transfer Required | ✅ | Enabled by default |
| 3.2 Infrastructure Encryption Enabled | ❌ | Accepted risk — lab environment |
| 3.7 Public Access Level Disabled | ✅ | Remediated — public access disabled |
| 3.8 Default Network Access Rule Set to Deny | ✅ | Remediated — network access disabled |
| 3.9 Allow Azure Trusted Services | ✅ | |
| 3.10 Private Endpoints Used | ❌ | Accepted risk — private endpoint deferred |
| 3.12 Customer Managed Keys | ❌ | Accepted risk — MMK sufficient for lab |
| 3.15 Minimum TLS Version 1.2 | ✅ | Correctly configured at deployment |

**Category 5 — Logging and Monitoring: FAILING ❌**

| Control | Status | Reason |
|---|---|---|
| 5.1 Configuring Diagnostic Settings | ❌ | Planned — Week 3 Sentinel integration |
| 5.2 Monitoring using Activity Log Alerts | ❌ | Planned — Week 3 Sentinel detection rules |
| 5.4 Azure Monitor Resource Logging | ✅ | Log Analytics workspace connected |

### 4.3 Category Analysis

**Category 1 — Identity and Access Management: PASSING ✅**

This is the most significant result. CIS Category 1 covers MFA enforcement, privileged access controls, and identity governance — exactly what was built in Week 1. The passing status directly validates:
- CA01/CA02: MFA enforcement for all users and admins
- CA03: Legacy authentication blocked
- PIM: JIT privileged access with no standing privilege
- Access Reviews: Periodic revalidation of privileged access

The identity governance program built in Week 1 is CIS-compliant.

**Category 2 — Microsoft Defender: FAILING ❌**

CIS requires enabling Microsoft Defender plans for all resource types. These plans are intentionally disabled in this lab environment to avoid unnecessary cost — no servers, databases, or containers are deployed. Three exemptions were created in Defender for Cloud with documented business justification. This is an accepted risk, not an unmanaged gap.

**Category 3 — Storage Accounts: FAILING ❌**

CIS requires storage accounts to use private endpoints (private link connections) to eliminate public internet exposure. The single storage account `fatiulabstorage01` does not have a private endpoint configured — this is an accepted risk documented in the Defender for Cloud remediation plan. All other storage account controls (shared key access disabled, public network access disabled) are passing.

**Category 5 — Logging and Monitoring: FAILING ❌**

CIS requires diagnostic settings and Log Analytics agents to be deployed on resources. The Log Analytics workspace `LAW-FatiuLab-Security` has been created and Sentinel data connectors are connected, but the Log Analytics agent has not been deployed to individual resources. This is a planned remediation item.

---

## 5. Microsoft Cloud Security Benchmark Results

The Microsoft Cloud Security Benchmark (MCSB) is automatically assessed by Defender for Cloud and reflects the post-remediation state.

| MCSB Category | Status |
|---|---|
| NS. Network Security | ❌ Private link finding |
| IM. Identity Management | ✅ Passing |
| PA. Privileged Access | ❌ Permanent GA — exempted |
| DP. Data Protection | ✅ Passing |
| AM. Asset Management | ✅ Passing |
| LT. Logging and Threat Detection | ✅ Passing |
| IR. Incident Response | ✅ Passing |
| PV. Posture and Vulnerability Management | ✅ Passing |
| ES. Endpoint Security | ✅ Passing |
| BR. Backup and Recovery | ✅ Passing |
| DS. DevOps Security | ✅ Passing |
| GS. Governance and Strategy | ⚪ Not evaluated |

**61 of 63 controls passing (97%)** — 2 remaining findings are accepted risks with documented exemptions.

Evidence: [P3-AZP-03-CIS-Compliance-Dashboard.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-03-CIS-Compliance-Dashboard.png)

---

## 6. Azure CSPM Compliance

The Azure CSPM standard tracks Defender CSPM-specific controls.

| Result | Value |
|---|---|
| Controls passing | 1 of 1 |
| Pass rate | 100% |

Evidence: [P3-AZP-04-CIS-Compliance-CSPM-Tab.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-04-CIS-Compliance-CSPM-Tab.png)

---

## 7. Log Streaming Policy

The Azure Activity log streaming policy was assigned as part of the Sentinel data connector configuration — ensuring all Azure subscription-level events are streamed to the Log Analytics workspace and available for Sentinel detection rules.

| Setting | Value |
|---|---|
| Policy | Configure Azure Activity logs to stream to specified Log Analytics workspace |
| Scope | Azure subscription 1 |
| Target workspace | LAW-FatiuLab-Security |
| Type | Azure Policy assignment |
| Purpose | Feed Azure Activity logs to Sentinel for detection engineering |

This policy ensures that every Azure management action — resource creation, deletion, role assignment, policy changes — is captured in Sentinel and available for KQL detection rules.

---

## 8. Compliance Remediation Plan

### 8.1 CIS Failing Categories — Remediation Approach

| Category | Finding | Action | Priority |
|---|---|---|---|
| 2. Microsoft Defender | CWPP plans disabled | Accepted risk — exempted with justification | Accepted |
| 3. Storage Accounts | Private link not configured | Accepted risk — lab environment | Accepted |
| 5. Logging and Monitoring | Log Analytics agent not deployed | Deploy diagnostic settings — Week 3 | Medium |

### 8.2 Target Compliance State

After deploying diagnostic settings in Week 3:

| Standard | Current | Target |
|---|---|---|
| CIS Azure Foundations v2.0.0 | 66/81 (81%) | 70/81 (86%+) |
| Microsoft Cloud Security Benchmark | 61/63 (97%) | 61/63 (97%) |
| Azure CSPM | 1/1 (100%) | 1/1 (100%) |

---

## 9. Gaps and Planned Improvements

| Gap | Risk | Planned Remediation | Priority |
|---|---|---|---|
| Log Analytics agent not deployed | Logging and monitoring CIS category failing | Deploy diagnostic settings in Week 3 | Medium |
| CIS initiative in Audit mode | Non-compliant resources not blocked | Move to Enforce mode after initial hardening | Medium |
| No NIST SP 800-53 initiative assigned | No federal framework coverage | Add NIST initiative for broader compliance | Low |
| No custom policy definitions | Only built-in policies assigned | Create custom policies for FatiuLab-specific controls | Low |
| No policy compliance alerts | Compliance drift not alerted | Configure Azure Monitor alerts on compliance state change | Low |

---

## 10. Compliance Alignment

| Control | PCI-DSS | PIPEDA | CIS v2.0.0 | NIST CSF |
|---|---|---|---|---|
| CIS benchmark assignment | Req 6.3 | Principle 7 | All categories | ID.GV-1 |
| Identity controls passing | Req 7.2, 8.4 | Principle 7 | Category 1 | PR.AC-1 |
| Storage account hardening | Req 6.2 | Principle 7 | Category 3 | PR.PT-3 |
| Log streaming policy | Req 10.2 | Principle 4 | Category 5 | DE.CM-1 |
| Continuous compliance monitoring | Req 6.3, 11.3 | Principle 7 | All | ID.RA-1 |

---

## 11. Evidence Index

| File | Description |
|---|---|
| [P3-AZP-00-Subscription-Linked.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-00-Subscription-Linked.png) | Azure subscription linked to FatiuLab Security tenant |
| [P3-AZP-01-CIS-Policy-Assignment.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-01-CIS-Policy-Assignment.png) | CIS initiative assignment — review page |
| [P3-AZP-02-CIS-Policy-Created.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-02-CIS-Policy-Created.png) | Policy assignments list — all 3 assignments |
| [P3-AZP-03-CIS-Compliance-Dashboard.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-03-CIS-Compliance-Dashboard.png) | Regulatory compliance dashboard — MCSB category breakdown |
| [P3-AZP-04-CIS-Compliance-CSPM-Tab.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-04-CIS-Compliance-CSPM-Tab.png) | Azure CSPM compliance — 1/1 passing |
| [P3-AZP-05-CIS-Benchmark-Results.png](../Evidence/Phase-02-Cloud-Security/Azure-Policy/P3-AZP-05-CIS-Benchmark-Results.png) | CIS Azure Foundations v2.0.0 — 66/81 controls, category breakdown |

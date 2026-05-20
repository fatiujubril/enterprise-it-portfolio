# Defender for Cloud — Cloud Security Posture Management
## FatiuLab Security — Azure Security Baseline & Remediation Program

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Regulatory Context:** PCI-DSS, PIPEDA  
**Tenant:** FatiuLabSecurity.onmicrosoft.com  
**Azure Subscription:** Azure subscription 1 (7b044f9e-6d7a-4ded-99b9-26b1561664c2)  
**Policy Author:** Global Administrator  
**Last Reviewed:** May 2026  
**Status:** Active — Phase 2 In Progress

---

## 1. Overview

Microsoft Defender for Cloud is the cloud-native security posture management (CSPM) and cloud workload protection platform (CWPP) for the FatiuLab Security Azure environment. It provides continuous assessment of the cloud environment against security best practices, generates prioritized recommendations, tracks a Secure Score, and measures regulatory compliance.

In a FinTech environment, cloud security posture management is not optional — PCI-DSS Requirement 6 mandates that all system components are protected and that security configurations are maintained. Defender for Cloud operationalizes this requirement by continuously assessing every deployed resource and surfacing misconfigurations before they can be exploited.

**What Defender for Cloud does:**

```
Deployed resource (e.g. storage account)
        ↓
Defender for Cloud scans resource configuration
        ↓
Compares configuration against security benchmarks
(Microsoft Cloud Security Benchmark, CIS, NIST)
        ↓
Generates prioritized recommendations
        ↓
Secure Score calculated based on healthy vs unhealthy controls
        ↓
Engineer remediates findings
        ↓
Secure Score improves — posture documented
```

---

## 2. Architecture

### 2.1 Subscription Scope

| Component | Value |
|---|---|
| Subscription | Azure subscription 1 |
| Subscription ID | 7b044f9e-6d7a-4ded-99b9-26b1561664c2 |
| Tenant | FatiuLab Security (FatiuLabSecurity.onmicrosoft.com) |
| Region | Canada Central |
| Owner | globaladmin@FatiuLabSecurity.onmicrosoft.com |

### 2.2 Defender Plans Enabled

| Plan | Type | Pricing | Status | Rationale |
|---|---|---|---|---|
| Foundational CSPM | CSPM | Free | On (always on) | Baseline posture assessment — always enabled |
| Defender CSPM | CSPM | $5/billable resource/month | On | Advanced posture — attack path analysis, cloud security explorer |
| Defender for Servers | CWPP | $15/server/month | Off | No servers deployed — cost avoided |
| Defender for Storage | CWPP | $10/storage account/month | Off | Cost avoided — CSPM covers posture assessment |
| Defender for Databases | CWPP | Variable | Off | No databases deployed |
| Defender for Containers | CWPP | Variable | Off | No containers deployed |
| All other CWPP plans | CWPP | Variable | Off | No applicable resources |

**Plan selection rationale:**

Foundational CSPM is always-on and free — it provides Secure Score, recommendations, and posture assessment for all resources. Defender CSPM adds attack path analysis and cloud security explorer at $5 per billable resource per month. With zero billable resources currently deployed the cost is $0, making the full CSPM experience available at no cost during this lab phase.

All CWPP (workload protection) plans are deliberately disabled. CWPP plans provide runtime threat detection for specific resource types and charge per resource deployed. Since this lab environment has no servers, databases, or containers, enabling CWPP plans would provide no protection benefit and incur unnecessary cost.

Evidence:
- [P3-DFC-01-Defender-Plans-Before.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-01-Defender-Plans-Before.png) — baseline state before enabling Defender CSPM
- [P3-DFC-02-Defender-Plans-Enabled.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-02-Defender-Plans-Enabled.png) — Defender CSPM enabled, full monitoring coverage

---

## 3. Resource Inventory

### 3.1 Deployed Resources

| Resource | Type | Resource Group | Region | Defender Plan |
|---|---|---|---|---|
| Azure subscription 1 | Subscription | N/A | Global | Foundational CSPM + Defender CSPM |
| fatiulabstorage01 | Storage account | RG-FatiuLab-Security | Canada Central | Off (CWPP) |
| Global Admin | Azure IAM user | N/A | Global | N/A |

**Total resources assessed:** 3  
**Unhealthy resources:** 3  
**Healthy resources:** 0

All three resources have active recommendations — this is the expected baseline state for a newly deployed environment with default configurations.

Evidence:
- [P3-DFC-03-Resource-Group.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-03-Resource-Group.png) — RG-FatiuLab-Security created
- [P3-DFC-04-Storage-Account-Config.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-04-Storage-Account-Config.png) — storage account insecure default configuration (before state)
- [P3-DFC-05-Storage-Account-Deployed.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-05-Storage-Account-Deployed.png) — storage account deployment confirmed
- [P3-DFC-06-Storage-Account-Overview.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-06-Storage-Account-Overview.png) — storage account properties
- [P3-DFC-07-Inventory.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-07-Inventory.png) — full inventory — 3 resources, 3 unhealthy

### 3.2 Storage Account — Insecure Default Configuration

The storage account `fatiulabstorage01` was deployed with Azure default settings. These defaults represent the baseline misconfiguration state that Defender for Cloud will assess and flag:

| Setting | Default Value | Security Risk | Expected Defender Recommendation |
|---|---|---|---|
| Storage account key access | Enabled | Full access bypasses Entra ID RBAC | Disable storage account key access |
| Default to Entra authorization | Disabled | Portal defaults to key-based access | Enable Entra ID as default authorization |
| Public network access | Enabled from all networks | Storage accessible from internet | Restrict to specific VNets or private endpoint |
| Infrastructure encryption | Disabled | Single layer encryption only | Enable infrastructure encryption |
| Minimum TLS version | 1.2 | Correctly configured | No recommendation expected |
| Blob anonymous access | Disabled | Correctly configured | No recommendation expected |

This intentional baseline misconfiguration provides real findings for the CSPM remediation story — demonstrating the full cycle from insecure deployment through assessment, prioritization, and remediation.

---

## 4. Secure Score Baseline

### 4.1 Baseline Metrics (May 2026)

| Metric | Value |
|---|---|
| Total Secure Score | **50%** |
| Azure Secure Score | 50% |
| Critical recommendations | 0 |
| High recommendations | 0 |
| Medium recommendations | 0 |
| Low recommendations | 10 |
| Active attack paths | 0 |
| Overdue recommendations | 0 |
| Assessed resources | 3 |
| Unhealthy resources | 3 |
| Regulatory compliance | 54 of 63 controls passed |
| Workload protection coverage | 0% (intentional) |

Evidence: [P3-DFC-08-Secure-Score-Baseline.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-08-Secure-Score-Baseline.png)

### 4.2 Score Interpretation

A baseline score of 50% on a fresh subscription with a single storage account is expected. The score reflects:

- 10 low-severity recommendations across the subscription and storage account
- No critical or high findings — the environment has no exposed services or high-risk configurations beyond default settings
- The subscription itself generates recommendations related to identity and monitoring configuration
- The Global Admin IAM user generates identity-related recommendations

**Target score after remediation:** 70%+ — achievable by remediating the top storage account and subscription-level findings documented in Section 5.

### 4.3 Regulatory Compliance Baseline

| Standard | Controls Passed | Controls Failed | Pass Rate |
|---|---|---|---|
| Microsoft Cloud Security Benchmark | 54 of 63 | 9 | 86% |
| Azure CSPM | 0 of 1 | 1 | 0% |

The Microsoft Cloud Security Benchmark is the primary compliance standard assessed by Defender for Cloud. 54 of 63 controls passing at baseline is a reasonable starting point — the 9 failing controls map directly to the recommendations we will remediate.

---

## 5. Recommendations — Baseline Findings

The following findings were identified by Defender for Cloud at baseline. These are the target remediation items for the Week 2 CSPM hardening exercise.

### 5.1 Storage Account Findings (fatiulabstorage01)

| Finding | Severity | Resource | Remediation |
|---|---|---|---|
| Storage account key access should be disabled | Medium | fatiulabstorage01 | Disable key access — enforce Entra ID RBAC |
| Default action for storage accounts should be set to deny | Medium | fatiulabstorage01 | Set network default action to Deny |
| Public network access should be disabled for storage accounts | Medium | fatiulabstorage01 | Disable public network access |
| Storage accounts should use customer-managed keys | Low | fatiulabstorage01 | Configure CMK encryption |
| Secure transfer to storage accounts should be enabled | Low | fatiulabstorage01 | Already enabled — verify |

### 5.2 Subscription-Level Findings

| Finding | Severity | Resource | Remediation |
|---|---|---|---|
| Subscriptions should have a contact email for security issues | Low | Subscription | Configure security contact email |
| Email notification for high severity alerts should be enabled | Low | Subscription | Enable alert notifications |
| MFA should be enabled for accounts with owner permissions | Low | Subscription | Enforced via CA02 — verify linkage |
| Log Analytics agent should be installed | Low | Subscription | Deploy monitoring agent |

### 5.3 Identity Findings (Global Admin IAM User)

| Finding | Severity | Resource | Remediation |
|---|---|---|---|
| There should be more than one owner assigned to subscription | Low | Global Admin | Add secondary owner or review |
| Guest accounts with owner permissions should be removed | Low | Global Admin | Verify no guest accounts with owner |

---

## 6. Remediation Plan

Remediation is prioritized by severity and business impact. All medium findings are addressed first, followed by high-impact low findings.

### Phase 1 — Storage Account Hardening (Priority: Medium findings first)

**Remediation 1 — Disable public network access**
```
Azure portal → fatiulabstorage01 → Networking
→ Public network access: Disabled
→ Save
```
Impact: Eliminates internet-accessible attack surface on storage account.

**Remediation 2 — Set network default action to Deny**
```
Azure portal → fatiulabstorage01 → Networking
→ Firewall and virtual networks
→ Default action: Deny
→ Save
```
Impact: All network traffic blocked by default — explicit allow rules required.

**Remediation 3 — Disable storage account key access**
```
Azure portal → fatiulabstorage01 → Configuration
→ Allow storage account key access: Disabled
→ Save
```
Impact: Forces all access through Entra ID RBAC — eliminates shared key bypass of identity controls.

### Phase 2 — Subscription Configuration (Priority: Low findings)

**Remediation 4 — Configure security contact**
```
Defender for Cloud → Environment settings → Azure subscription 1
→ Email notifications
→ Add security contact email
→ Save
```

**Remediation 5 — Enable high severity alert notifications**
```
Defender for Cloud → Environment settings → Azure subscription 1
→ Email notifications
→ Notify about alerts with severity: High
→ Save
```

### 6.1 Before/After Secure Score

| Metric | Before Remediation | After Remediation (Target) |
|---|---|---|
| Secure Score | 50% | 70%+ |
| Medium recommendations | 0 active (storage findings) | 0 |
| Low recommendations | 10 | 5 or fewer |
| Regulatory compliance | 54/63 | 58/63+ |

*After screenshots will be added upon remediation completion.*

---

## 7. Defender for Cloud Integration with Identity Controls

Defender for Cloud does not operate in isolation — it integrates with the identity controls built in Week 1:

| Integration | How It Works |
|---|---|
| CA04 + Defender for Cloud | CA04 requires MFA for Azure Management access — prevents unauthorized changes to Defender settings |
| PIM + Defender for Cloud | Security Admin must activate role via JIT before making Defender configuration changes — all changes are audited |
| Entra ID RBAC | Storage account remediation enforces Entra ID authorization — aligns with Zero Trust identity-first access model |
| Access Reviews | Global Admin IAM user finding is monitored via AR-01 quarterly review |

---

## 8. Gaps and Planned Improvements

| Gap | Risk | Planned Remediation | Priority |
|---|---|---|---|
| No Azure Policy initiative assigned | Compliance drift not prevented | Assign CIS or NIST benchmark policy initiative — azure-policy.md | High |
| No Log Analytics workspace | Diagnostic logs not collected | Deploy Log Analytics + connect to Sentinel — Week 3 | High |
| No alert notifications configured | Security findings not notified | Configure security contact and email alerts | Medium |
| Workload protection 0% | No runtime threat detection | Accept for lab — document as known gap | Low |
| No private endpoint for storage | Storage accessible via public endpoint | Implement private endpoint in maturity roadmap | Medium |

---

## 9. Compliance Alignment

| Control | PCI-DSS | PIPEDA | NIST CSF | Zero Trust |
|---|---|---|---|---|
| Continuous posture assessment | Req 6.3, 11.3 | Principle 7 | ID.RA-1 | Assume breach |
| Secure Score tracking | Req 6.3 | Principle 7 | ID.RA-1 | Assume breach |
| Storage account hardening | Req 6.2, 7.2 | Principle 7 | PR.PT-3 | Least privilege |
| Network access restriction | Req 1.3 | Principle 7 | PR.AC-3 | Assume breach |
| Key access disabled | Req 8.2 | Principle 7 | PR.AC-1 | Verify explicitly |
| Security contact configured | Req 12.10 | Principle 7 | RS.CO-2 | Assume breach |

---

## 10. Evidence Index

| File | Description |
|---|---|
| [P3-DFC-01-Defender-Plans-Before.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-01-Defender-Plans-Before.png) | Baseline — all plans off except Foundational CSPM |
| [P3-DFC-02-Defender-Plans-Enabled.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-02-Defender-Plans-Enabled.png) | Defender CSPM enabled — full monitoring coverage |
| [P3-DFC-03-Resource-Group.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-03-Resource-Group.png) | RG-FatiuLab-Security created |
| [P3-DFC-04-Storage-Account-Config.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-04-Storage-Account-Config.png) | Storage account insecure defaults — before state |
| [P3-DFC-05-Storage-Account-Deployed.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-05-Storage-Account-Deployed.png) | Storage account deployment confirmed |
| [P3-DFC-06-Storage-Account-Overview.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-06-Storage-Account-Overview.png) | Storage account properties overview |
| [P3-DFC-07-Inventory.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-07-Inventory.png) | Inventory — 3 resources, 3 unhealthy |
| [P3-DFC-08-Secure-Score-Baseline.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-08-Secure-Score-Baseline.png) | Secure Score baseline — 50% |
| P3-DFC-09-Recommendations.png | Recommendations list — pending screenshot |
| P3-DFC-10-Secure-Score-After.png | Secure Score after remediation — pending |

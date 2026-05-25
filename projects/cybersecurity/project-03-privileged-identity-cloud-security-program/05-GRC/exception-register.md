# Exception Register
## FatiuLab Security — Security Control Exception & Risk Acceptance Register

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Regulatory Context:** PCI-DSS, PIPEDA  
**Register Owner:** Global Administrator  
**Last Reviewed:** May 2026  
**Review Frequency:** Quarterly

---

## 1. Overview

This register documents all approved exceptions to the FatiuLab Security security control baseline. An exception is a formal acknowledgment that a control cannot be fully implemented as designed, along with the business justification, compensating controls, and accepted residual risk.

**Exception policy:**

> No security control gap is acceptable without a formal exception. Every exception must have a documented business justification, an identified risk owner, compensating controls, and a defined review date. Exceptions without these elements are treated as unmitigated risks.

**Exception lifecycle:**

```
Control gap identified
        ↓
Risk owner documents exception
        ↓
Justification reviewed and approved
        ↓
Compensating controls implemented
        ↓
Exception recorded in this register
        ↓
Quarterly review — still valid?
        ↓
Yes → renew          No → remediate or close
```

---

## 2. Exception Register

---

### EXP-01 — Permanent Global Administrator Assignment

**Exception ID:** EXP-01  
**Created:** May 2026  
**Status:** Active  
**Next review:** August 2026

| Field | Value |
|---|---|
| Control violated | PIM — no permanent active assignments |
| MCSB finding | PA. Privileged Access — failing |
| Defender for Cloud finding | "Privileged roles should not have permanent access" |
| Risk owner | Global Administrator |
| Business justification | One permanent Global Administrator assignment is required as a PIM safety anchor. Without a permanent GA, if PIM has an outage or a bug prevents role activation, all privileged access would be blocked and tenant recovery would be impossible. The permanent GA is the last-resort recovery mechanism for the tenant. |
| Affected account | globaladmin@FatiuLabSecurity.onmicrosoft.com |

**Risk assessment:**

| Field | Value |
|---|---|
| Inherent risk without exception | High — permanent GA is highest-risk standing privilege |
| Residual risk with compensating controls | Medium — controls significantly reduce exposure |

**Compensating controls:**

| Control | Description |
|---|---|
| MFA enforced | CA01 and CA02 require MFA for all Global Admin access |
| PIM monitoring | DET-02 detects any additional GA assignments outside PIM |
| Break-glass separation | Break-glass account is a separate account — GA permanent assignment is on primary admin account only |
| Audit logging | All Global Admin activity logged to Sentinel |
| Access review | AR-01 quarterly review covers SEC-Global-Admins group |

**Exemption in Defender for Cloud:** EXP-01-PIM-Permanent-GA-Intentional

**Evidence:** [P3-DFC-13-Exemptions.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-13-Exemptions.png)

---

### EXP-02 — Microsoft Defender for Resource Manager Not Enabled

**Exception ID:** EXP-02  
**Created:** May 2026  
**Status:** Active  
**Next review:** August 2026

| Field | Value |
|---|---|
| Control violated | CIS 2.1.12 — Defender for Resource Manager should be enabled |
| MCSB finding | PV. Posture and Vulnerability Management |
| Defender for Cloud finding | "Microsoft Defender for Resource Manager should be enabled" |
| Pricing | $5/subscription/month |
| Risk owner | Global Administrator |
| Business justification | Defender for Resource Manager provides runtime threat detection for Azure Resource Manager operations. In this lab environment, all Azure Management access is protected by CA04 (MFA required) and monitored via DET-06 (Azure role assignment detection) and DET-08 (Defender findings). The $5/month cost is not justified given the existing compensating controls and minimal resource deployment. |

**Risk assessment:**

| Field | Value |
|---|---|
| Inherent risk without exception | Medium — ARM operations not runtime-monitored |
| Residual risk with compensating controls | Low — compensating controls provide adequate coverage |

**Compensating controls:**

| Control | Description |
|---|---|
| CA04 | MFA required for all Azure Management access |
| DET-06 | Azure privileged role assignment detection |
| DET-08 | Defender for Cloud high severity finding detection |
| AzureActivity logs | All ARM operations captured in Sentinel |
| CIS benchmark | Continuous compliance monitoring |

**Exemption in Defender for Cloud:** EXP-02-Defender-ResourceManager-Cost-Avoided

**Evidence:** [P3-DFC-13-Exemptions.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-13-Exemptions.png)

---

### EXP-03 — Microsoft Defender for Storage Not Enabled

**Exception ID:** EXP-03  
**Created:** May 2026  
**Status:** Active  
**Next review:** August 2026

| Field | Value |
|---|---|
| Control violated | CIS 2.1.7 — Defender for Storage should be enabled |
| Pricing | $10/storage account/month + $0.15/GB scanned |
| Risk owner | Global Administrator |
| Business justification | Defender for Storage provides malware scanning and data sensitivity detection for Azure Blob Storage. The single lab storage account fatiulabstorage01 contains no sensitive data — it was deployed purely to generate CSPM findings for the portfolio. Public network access is disabled, shared key access is disabled, and the account is protected by existing CSPM controls. The cost is not justified for an empty lab storage account. |

**Risk assessment:**

| Field | Value |
|---|---|
| Inherent risk without exception | Low — no sensitive data in storage account |
| Residual risk with compensating controls | Low — network isolation and CSPM provide adequate coverage |

**Compensating controls:**

| Control | Description |
|---|---|
| Public network access disabled | Storage account not accessible from internet |
| Shared key access disabled | All access via Entra ID RBAC only |
| Defender CSPM | Continuous posture assessment for storage configuration |
| No sensitive data | Storage account contains no payment or customer data |

**Exemption in Defender for Cloud:** EXP-03-Defender-Storage-Cost-Avoided

**Evidence:** [P3-DFC-13-Exemptions.png](../Evidence/Phase-02-Cloud-Security/Defender-for-Cloud/P3-DFC-13-Exemptions.png)

---

### EXP-04 — Storage Account Private Endpoint Not Configured

**Exception ID:** EXP-04  
**Created:** May 2026  
**Status:** Active  
**Next review:** August 2026

| Field | Value |
|---|---|
| Control violated | CIS 3.10 — Private endpoints should be used for storage accounts |
| Defender for Cloud finding | "Storage account should use a private link connection" |
| Risk owner | Global Administrator |
| Business justification | Private endpoints require an Azure Virtual Network to be deployed, which adds complexity and cost beyond the scope of this lab phase. The storage account's network access is disabled entirely — it is not accessible from any public or private network. This is a more restrictive control than a private endpoint, which would restrict access to a specific VNet while still allowing network access. |

**Risk assessment:**

| Field | Value |
|---|---|
| Inherent risk without exception | Medium — no private endpoint isolation |
| Residual risk with compensating controls | Low — complete network isolation is more restrictive than private endpoint |

**Compensating controls:**

| Control | Description |
|---|---|
| Public network access disabled | More restrictive than private endpoint — all network access blocked |
| Shared key access disabled | No key-based access possible |
| No sensitive data | Storage account contains no production data |

**Note:** Public network access disabled is actually a stronger control than private endpoint in isolation — it blocks all network access rather than restricting to a specific VNet. This exception is primarily for CIS benchmark compliance purposes.

---

### EXP-05 — Infrastructure Encryption Not Enabled

**Exception ID:** EXP-05  
**Created:** May 2026  
**Status:** Active  
**Next review:** August 2026

| Field | Value |
|---|---|
| Control violated | CIS 3.2 — Infrastructure encryption should be enabled for storage accounts |
| Risk owner | Global Administrator |
| Business justification | Infrastructure encryption (double encryption) adds a second layer of AES-256 encryption at the hardware level in addition to the default service-level encryption. This provides protection against physical compromise of storage hardware. For a lab environment with no sensitive data, the additional cost and complexity of infrastructure encryption is not justified. Standard Azure-managed encryption (MMK) provides adequate protection for lab workloads. |

**Risk assessment:**

| Field | Value |
|---|---|
| Inherent risk without exception | Low — single-layer encryption still strong |
| Residual risk with compensating controls | Low — MMK encryption adequate for lab data |

**Compensating controls:**

| Control | Description |
|---|---|
| Azure-managed encryption (MMK) | AES-256 encryption at rest — enabled by default |
| Secure transfer required | TLS 1.2 encryption in transit |
| No sensitive data | No production or payment data stored |

---

### EXP-06 — Customer Managed Keys Not Configured

**Exception ID:** EXP-06  
**Created:** May 2026  
**Status:** Active  
**Next review:** August 2026

| Field | Value |
|---|---|
| Control violated | CIS 3.12 — Storage accounts should use customer managed keys |
| Risk owner | Global Administrator |
| Business justification | Customer Managed Keys (CMK) allow the organization to control the encryption keys rather than using Microsoft-managed keys. CMK requires Azure Key Vault which adds cost and operational complexity. For a lab environment with no sensitive data, Microsoft-managed keys provide adequate encryption. CMK would be required in a production environment handling payment data under PCI-DSS. |

**Risk assessment:**

| Field | Value |
|---|---|
| Inherent risk without exception | Low — MMK encryption is strong |
| Residual risk with compensating controls | Low — adequate for lab environment |

**Compensating controls:**

| Control | Description |
|---|---|
| Azure-managed encryption (MMK) | AES-256 encryption — Microsoft manages keys |
| No sensitive data | No production data requiring CMK |

**Production note:** In a production environment handling payment data, CMK would be required to satisfy PCI-DSS Requirement 3.7 (protect cryptographic keys). This exception is lab-specific and would not be acceptable in production.

---

### EXP-07 — CA05 Standard Users Policy in Report-Only Mode

**Exception ID:** EXP-07  
**Created:** May 2026  
**Status:** Active — pending MFA registration campaign  
**Next review:** June 2026 (shorter cycle — active remediation)

| Field | Value |
|---|---|
| Control violated | CA05-Restrict-Standard-Users should be set to On |
| Risk owner | Global Administrator |
| Business justification | CA05 requires MFA for all standard users. The policy is in Report-only mode because standard users have not yet completed MFA registration. Enabling enforcement before registration is complete would lock users out — an availability incident. This is a staged rollout approach, not a permanent exception. |
| Target remediation date | June 2026 |

**Risk assessment:**

| Field | Value |
|---|---|
| Inherent risk without exception | Medium — standard users not MFA-enforced |
| Residual risk with compensating controls | Medium — CA01 still enforces MFA (all users) |

**Compensating controls:**

| Control | Description |
|---|---|
| CA01 | MFA required for all users — covers standard users during transition |
| CA03 | Legacy auth blocked for all users including standard users |
| Report-only monitoring | Sign-in logs reviewed for policy impact |

**Remediation plan:**
1. Complete MFA registration campaign for all standard users
2. Confirm registration via Entra ID Authentication Methods report
3. Switch CA05 from Report-only to On
4. Close this exception

---

## 3. Exception Summary

| ID | Exception | Status | Risk Level | Review Date |
|---|---|---|---|---|
| EXP-01 | Permanent GA Assignment | Active | Medium | Aug 2026 |
| EXP-02 | Defender for Resource Manager | Active | Low | Aug 2026 |
| EXP-03 | Defender for Storage | Active | Low | Aug 2026 |
| EXP-04 | Storage Private Endpoint | Active | Low | Aug 2026 |
| EXP-05 | Infrastructure Encryption | Active | Low | Aug 2026 |
| EXP-06 | Customer Managed Keys | Active | Low | Aug 2026 |
| EXP-07 | CA05 Report-Only | Active | Medium | Jun 2026 |

**Total exceptions: 7**
**High or Critical exceptions: 0**
**Exceptions requiring immediate attention: 0**

---

## 4. Exception Governance

**Approval authority:**
- Low risk exceptions: Global Administrator
- Medium risk exceptions: Global Administrator + Security Administrator review
- High risk exceptions: Global Administrator + external security review required
- Critical risk exceptions: Not permitted — must remediate

**Review cadence:**
- All exceptions reviewed quarterly
- Medium risk exceptions reviewed more frequently if compensating controls change
- Exception closed when control is fully implemented or risk no longer applies

**Exception aging policy:**
- Exceptions older than 12 months without renewal are automatically escalated
- Exceptions with no compensating controls are treated as unmitigated risks

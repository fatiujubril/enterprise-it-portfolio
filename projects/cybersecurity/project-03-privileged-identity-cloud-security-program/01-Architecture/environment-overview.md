# Environment Overview
## FatiuLab Security — Lab Environment Documentation

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Tenant:** FatiuLabSecurity.onmicrosoft.com  
**Last Updated:** May 2026

---

## 1. Tenant Overview

| Component | Detail |
|---|---|
| Tenant name | FatiuLab Security |
| Primary domain | FatiuLabSecurity.onmicrosoft.com |
| License | Microsoft 365 Business Premium |
| Entra ID tier | P2 (included in Business Premium) |
| Region | Canada Central |
| Created | May 2026 |

Evidence: [P3-BL-01-Tenant-Overview.png](../Evidence/Phase-01-Identity/P3-BL-01-Tenant-Overview.png)

---

## 2. User Accounts

| Display Name | UPN | Role | Purpose |
|---|---|---|---|
| Global Admin | globaladmin@FatiuLabSecurity.onmicrosoft.com | Global Administrator (permanent) | Primary admin — tenant management |
| Security Admin | securityadmin@FatiuLabSecurity.onmicrosoft.com | No permanent role — PIM eligible | Security operations — JIT access only |
| Standard User | standarduser@FatiuLabSecurity.onmicrosoft.com | No admin role | Standard user — CA policy testing |
| Break Glass | breakglass@FatiuLabSecurity.onmicrosoft.com | Global Administrator (permanent) | Emergency access — no MFA registered |

Evidence: [P3-BL-03-Users-List.png](../Evidence/Phase-01-Identity/P3-BL-03-Users-List.png)

---

## 3. Security Groups

| Group Name | Type | Members | Purpose |
|---|---|---|---|
| SEC-Global-Admins | Security | globaladmin@, breakglass@ | Global admin group |
| SEC-Admins | Security | securityadmin@ | Security admin group |
| SEC-Standard-Users | Security | standarduser@ | Standard user population |
| SEC-CA-Exclusions | Security | breakglass@ only | CA policy exclusion — break-glass only |
| SEC-PIM-Eligible | Security | securityadmin@ | PIM eligible role assignments |

Evidence: [P3-BL-04-Security-Groups.png](../Evidence/Phase-01-Identity/P3-BL-04-Security-Groups.png)

---

## 4. Licensing

| License | Assigned To | Key Features |
|---|---|---|
| Microsoft 365 Business Premium | All users | Exchange Online, Teams, SharePoint |
| Microsoft Entra ID P2 | All users | PIM, Identity Protection, Access Reviews, Conditional Access |

Evidence: [P3-BL-02-P2-License-Assigned.png](../Evidence/Phase-01-Identity/P3-BL-02-P2-License-Assigned.png)

---

## 5. Azure Subscription

| Component | Detail |
|---|---|
| Subscription name | Azure subscription 1 |
| Subscription ID | 7b044f9e-6d7a-4ded-99b9-26b1561664c2 |
| Directory | FatiuLab Security |
| Status | Active |
| Role | Owner (globaladmin@) |
| Region | Canada Central |

---

## 6. Azure Resources

| Resource | Type | Resource Group | Region | Purpose |
|---|---|---|---|---|
| RG-FatiuLab-Security | Resource Group | N/A | Canada Central | Container for all lab resources |
| fatiulabstorage01 | Storage Account | RG-FatiuLab-Security | Canada Central | CSPM findings generation + hardening |
| LAW-FatiuLab-Security | Log Analytics Workspace | RG-FatiuLab-Security | Canada Central | Sentinel data store |

---

## 7. Microsoft Sentinel

| Component | Detail |
|---|---|
| Workspace name | LAW-FatiuLab-Security |
| Workspace ID | e1030a74-426b-41b7-baf8-3cef510c9753 |
| Resource group | RG-FatiuLab-Security |
| Pricing tier | Pay-as-you-go |
| Data connectors | 11 connected |
| Analytics rules | 8 active |

---

## 8. Named Locations

| Name | Type | IP Range | Trusted |
|---|---|---|---|
| LOC-FatiuLab-HQ | IP ranges | 192.168.1.0/24 | Yes |

---

## 9. Environment Design Decisions

**Why a fresh tenant?**
A dedicated lab tenant isolates all configurations from any existing environment. There's no risk of breaking production controls, no leftover configurations from previous projects, and the entire build is documented from baseline — making it auditable and repeatable.

**Why M365 Business Premium?**
Business Premium includes Entra ID P2 which is required for PIM, Identity Protection, and Access Reviews — the core identity governance controls in this program. It also includes Defender for Business which integrates with the Microsoft security stack.

**Why Canada Central?**
PIPEDA compliance — Canada Central keeps data residency within Canadian jurisdiction, which is appropriate for a fictional Canadian FinTech.

**Why a single storage account?**
The storage account was deployed with intentional insecure defaults to generate meaningful CSPM findings. A single resource provides sufficient findings to demonstrate the full CSPM remediation workflow without adding unnecessary cost.

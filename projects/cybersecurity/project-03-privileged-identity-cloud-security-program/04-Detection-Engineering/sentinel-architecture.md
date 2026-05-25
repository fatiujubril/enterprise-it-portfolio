# Sentinel Architecture
## FatiuLab Security — Microsoft Sentinel SIEM Design & Configuration

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Workspace:** LAW-FatiuLab-Security  
**Last Updated:** May 2026

---

## 1. Overview

Microsoft Sentinel is the cloud-native SIEM (Security Information and Event Management) platform for FatiuLab Security. It aggregates logs from Microsoft Entra ID, Azure, and Microsoft Defender for Cloud into a single workspace, runs custom KQL analytics rules to detect threats, and creates incidents for security team response.

**Why Sentinel for this environment:**
- Native integration with the entire Microsoft security stack — no agent configuration needed for Entra ID and Azure logs
- KQL query language — same language used across all Microsoft security products
- Scales from a single storage account to an enterprise estate without architectural changes
- Incident management built in — alerts automatically become actionable incidents

---

## 2. Workspace Architecture

### 2.1 Log Analytics Workspace

All Sentinel data is stored in a Log Analytics workspace — the underlying data platform that Sentinel sits on top of.

| Component | Value |
|---|---|
| Workspace name | LAW-FatiuLab-Security |
| Workspace ID | e1030a74-426b-41b7-baf8-3cef510c9753 |
| Resource group | RG-FatiuLab-Security |
| Region | Canada Central |
| Pricing tier | Pay-as-you-go |
| Retention | 90 days (default) |

Evidence: [P3-SEN-01-LogAnalytics-Workspace.png](../Evidence/Phase-03-Detections/Sentinel/P3-SEN-01-LogAnalytics-Workspace.png)

### 2.2 Sentinel Workspace

Microsoft Sentinel is enabled on top of the Log Analytics workspace, adding SIEM capabilities — analytics rules, incidents, threat intelligence, workbooks, and automation.

Evidence: [P3-SEN-02-Sentinel-Workspace.png](../Evidence/Phase-03-Detections/Sentinel/P3-SEN-02-Sentinel-Workspace.png)

---

## 3. Data Connectors

11 data connectors are connected, streaming logs from across the Microsoft security stack into the Sentinel workspace.

Evidence: [P3-SEN-03-Data-Connectors.png](../Evidence/Phase-03-Detections/Sentinel/P3-SEN-03-Data-Connectors.png)

### 3.1 Primary Connectors (Used by Detection Rules)

| Connector | Tables Populated | Detection Rules |
|---|---|---|
| Microsoft Entra ID | AuditLogs, SigninLogs | DET-01, DET-02, DET-03, DET-04, DET-05, DET-07 |
| Azure Activity | AzureActivity | DET-06, DET-08 |
| Tenant-based Microsoft Defender for Cloud | AzureActivity (Security) | DET-08 |

### 3.2 Full Connector Inventory

| Connector | Status | Purpose |
|---|---|---|
| Azure Activity | Connected | Azure subscription operations |
| Microsoft 365 Insider Risk Management | Connected | Insider risk signals |
| Microsoft Defender for Cloud Apps | Connected | Cloud app activity |
| Microsoft Defender for Endpoint | Connected | Endpoint signals |
| Microsoft Defender for Identity | Connected | Identity threat signals |
| Microsoft Defender for Office 365 | Connected | Email threat signals |
| Microsoft Defender XDR | Connected | Unified XDR signals |
| Microsoft Entra ID | Connected | Sign-in and audit logs |
| Microsoft Entra ID Protection | Connected | Identity risk events |
| Subscription-based Microsoft Defender for Cloud (Legacy) | Connected | Legacy security alerts |
| Tenant-based Microsoft Defender for Cloud | Connected | Defender for Cloud alerts |

### 3.3 Microsoft Entra ID Connector Configuration

The Entra ID connector required explicit log type selection. The following log types are enabled:

| Log Type | Table | Detection Use |
|---|---|---|
| Sign-In Logs | SigninLogs | DET-05, DET-07 |
| Audit Logs | AuditLogs | DET-01, DET-02, DET-03, DET-04 |
| Non-Interactive Sign-In Logs | SigninLogs | Supplementary |
| Service Principal Sign-In Logs | SigninLogs | Supplementary |
| Provisioning Logs | AADProvisioningLogs | Future use |

### 3.4 Azure Activity Connector Configuration

Azure Activity uses an Azure Policy assignment to stream subscription-level logs to the workspace:

| Setting | Value |
|---|---|
| Policy | Configure Azure Activity logs to stream to specified Log Analytics workspace |
| Scope | Azure subscription 1 |
| Target workspace | LAW-FatiuLab-Security |
| Method | Diagnostic settings pipeline (new) |

---

## 4. Log Flow Architecture

```
Microsoft Entra ID
├── Sign-in events ──────────────────────────────► SigninLogs
│   (every authentication attempt)
└── Audit events ────────────────────────────────► AuditLogs
    (PIM activations, CA changes, user management)

Azure Resource Manager
└── Subscription operations ─────────────────────► AzureActivity
    (role assignments, resource deployments,
     policy changes, Defender alerts)

Microsoft Defender for Cloud
└── Security alerts ─────────────────────────────► AzureActivity
    (CSPM findings, threat detections)          (Security category)

All tables ──────────────────────────────────────► LAW-FatiuLab-Security
                                                   (Log Analytics Workspace)
                                                          │
                                                          ▼
                                                   Analytics Rules
                                                   (KQL evaluation)
                                                          │
                                                          ▼
                                                   Incidents
                                                   (security team response)
```

---

## 5. Analytics Rules Architecture

### 5.1 Rule Types

| Type | How It Works | Used For |
|---|---|---|
| Scheduled | Runs on defined schedule, queries a time window | Most rules — pattern detection over time |
| NRT (Near Real Time) | Evaluates new events within 1-2 minutes | Break-glass sign-in — immediate detection needed |
| Anomaly | ML-based behavioral detection | Not configured — future roadmap |

### 5.2 Rule Inventory

| ID | Rule Name | Type | Frequency | Data Source |
|---|---|---|---|---|
| DET-01 | PIM Activation Outside Business Hours | Scheduled | 1 hour | AuditLogs |
| DET-02 | Global Admin Assigned Outside PIM | Scheduled | 1 hour | AuditLogs |
| DET-03 | MFA Disabled for Privileged Account | Scheduled | 1 hour | AuditLogs |
| DET-04 | CA Policy Modified or Disabled | Scheduled | 1 hour | AuditLogs |
| DET-05 | Impossible Travel on Admin Account | Scheduled | 1 hour | SigninLogs |
| DET-06 | Azure Privileged Role Assignment | Scheduled | 1 hour | AzureActivity |
| DET-07 | Break-Glass Account Sign-In | NRT | ~1-2 min | SigninLogs |
| DET-08 | Defender for Cloud High Severity Finding | Scheduled | 1 hour | AzureActivity |

### 5.3 Incident Settings

All rules are configured with:
- **Create incidents from alerts:** Enabled
- **Alert grouping:** Enabled — related alerts grouped into single incident
- **Grouping period:** 5 hours

---

## 6. Data Verification

Before deploying analytics rules, all three primary tables were verified to contain data:

```kql
-- AuditLogs verification
AuditLogs
| where Category == "RoleManagement"
| project TimeGenerated, OperationName
| order by TimeGenerated desc
-- Result: PIM activation events confirmed ✅

-- SigninLogs verification
SigninLogs
| where UserPrincipalName contains "breakglass"
| project TimeGenerated, UserPrincipalName, AppDisplayName
-- Result: Break-glass sign-in events confirmed ✅

-- AzureActivity verification
AzureActivity
| where OperationNameValue contains "roleAssignments"
| project TimeGenerated, OperationNameValue, ActivityStatusValue
-- Result: Role assignment events confirmed ✅
```

Simulation activities were performed on May 24, 2026 to generate real events across all tables before rule deployment — ensuring rules could be validated against actual log data.

---

## 7. Sentinel Migration Note

As of May 2026, Microsoft is migrating Sentinel to the unified Microsoft Defender portal (security.microsoft.com). The migration deadline is March 31, 2027 — after which Sentinel will only be available through the Defender portal.

**Impact on this environment:** None for current functionality. The analytics rules, data connectors, and workspace configuration remain identical. The access path changes from portal.azure.com → Microsoft Sentinel to security.microsoft.com → Microsoft Sentinel.

---

## 8. Cost Considerations

| Component | Cost Model | Current Cost |
|---|---|---|
| Log Analytics ingestion | Pay-per-GB ingested | Minimal — lab environment, low volume |
| Log Analytics retention | First 90 days free | $0 |
| Sentinel | Pay-per-GB analyzed | Minimal — lab volume |
| NRT rules | Higher compute than Scheduled | Negligible at lab scale |

**Cost optimization applied:**
- CWPP plans disabled — no workload protection charges
- Single workspace — no multi-workspace overhead
- Minimal resource deployment — low log volume

---

## 9. Gaps and Planned Improvements

| Gap | Impact | Planned Remediation |
|---|---|---|
| No automation playbooks | Manual containment only | Deploy Logic Apps playbook for account disable |
| No threat intelligence | No IOC matching | Connect Microsoft Threat Intelligence connector |
| 90-day retention only | Limited historical investigation | Extend retention or archive to storage |
| No workbook configured | No visual dashboard | Build identity security workbook |
| Scheduled rules only (mostly) | Up to 1hr detection latency | Convert DET-02, DET-04, DET-06 to NRT |

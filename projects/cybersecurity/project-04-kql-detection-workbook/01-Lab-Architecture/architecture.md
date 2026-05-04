# Lab Architecture

## Overview
This lab uses Microsoft Sentinel deployed on an Azure Log Analytics workspace to ingest, monitor, and detect threats using custom KQL analytics rules.

---

## Components

| Component | Name | Details |
|---|---|---|
| Azure Tenant | Fatiu Lab | fatiulab.onmicrosoft.com |
| Azure Subscription | Azure subscription 1 | Pay-as-you-go |
| Resource Group | rg-sentinel-lab | Canada East |
| Log Analytics Workspace | law-sentinel-lab-fatiu | Canada East |
| SIEM | Microsoft Sentinel | Deployed on law-sentinel-lab-fatiu |

---

## Data Sources Connected

| Source | Table in Sentinel | License Required | Status |
|---|---|---|---|
| Azure Activity Logs | AzureActivity | None — free | 🟡 Connecting |
| Entra ID Audit Logs | AuditLogs | Entra ID Free | 🔴 Pending |
| Entra ID Sign-in Logs | SigninLogs | Entra ID P1 | 🔴 Pending |

---

## Cost Controls
- Daily ingestion cap: 2 GB/day
- Estimated actual cost: under 10 CAD/month after 31-day free trial

---

## Lab Setup Steps
1. Created Log Analytics Workspace law-sentinel-lab-fatiu in Canada East
2. Deployed Microsoft Sentinel on the workspace
3. Installed Azure Activity connector via Content Hub
4. Connected Azure Activity logs via Azure Policy Assignment Wizard

---

## Evidence
- [Sentinel workspace created](../Evidence/01-lab-setup/sentinel-workspace-created.png)
- [Data connectors page](../Evidence/01-lab-setup/data-connectors-enabled.png)
- [Logs flowing verification](../Evidence/01-lab-setup/logs-flowing-azureactivity.png)

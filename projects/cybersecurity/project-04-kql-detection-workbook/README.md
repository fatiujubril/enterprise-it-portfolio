# Project 04 — KQL Detection Workbook

## Overview
A hands-on detection engineering project built in Microsoft Sentinel. This workbook contains 10 custom KQL detection rules targeting real MITRE ATT&CK techniques observed in identity and cloud environments.

Each rule is documented with query logic, threshold rationale, test results, and tuning decisions — mirroring enterprise detection engineering workflows.

---

## Target Roles
- Security Operations Analyst (Travelopia)
- Information Security Engineer (Opensity Solutions)

---

## Environment
| Component | Details |
|---|---|
| SIEM | Microsoft Sentinel |
| Workspace | law-sentinel-lab-fatiu |
| Region | Canada East |
| Data Sources | Azure Activity Logs, Entra ID Audit Logs |
| Lab Tenant | Azure subscription 1 / Fatiu Lab |

---

## MITRE ATT&CK Coverage

| Rule | Technique | Tactic | Severity |
|---|---|---|---|
| Impossible Travel | T1078.004 | Initial Access | High |
| Brute Force Detection | T1110 | Credential Access | High |
| MFA Fatigue | T1621 | Credential Access | High |
| Suspicious App Consent | T1528 | Credential Access | Medium |
| New Admin Role Assignment | T1098 | Privilege Escalation | High |
| Bulk User Deletion | T1531 | Impact | High |
| Legacy Auth Sign-in | T1078 | Initial Access | Medium |
| After Hours Sign-in | T1078 | Initial Access | Low |
| Service Principal Anomaly | T1528 | Persistence | Medium |
| Account Manipulation | T1098.001 | Privilege Escalation | High |

---

## Project Structure

\\\

project-04-kql-detection-workbook/
├── 00-Executive-Summary/
├── 01-Lab-Architecture/
├── 02-MITRE-Coverage/
├── 03-Detection-Rules/
│   ├── T1078-valid-accounts/
│   ├── T1110-brute-force/
│   ├── T1621-mfa-fatigue/
│   ├── T1528-app-consent/
│   ├── T1098-account-manipulation/
│   ├── T1531-account-access-removal/
│   ├── T1078-legacy-auth/
│   ├── T1078-after-hours/
│   └── T1528-service-principal/
├── 04-Tuning-Log/
├── 05-Workbook-Export/
├── 06-Lessons-Learned/
└── Evidence/
    ├── 01-lab-setup/
    ├── 02-mitre-coverage/
    ├── 03-detection-rules/
    ├── 04-tuning/
    └── 05-workbook/
\\\

---

## Status
🟡 In Progress — Environment setup and log ingestion phase

---

## Author
Fatiu Jubril
[LinkedIn](https://www.linkedin.com/in/fatiu-jubril/)
[GitHub Portfolio](https://github.com/fatiujubril/enterprise-it-portfolio)

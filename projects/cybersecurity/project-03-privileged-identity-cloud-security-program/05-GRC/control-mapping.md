# Control Mapping
## FatiuLab Security — NIST CSF Control Framework Mapping

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Regulatory Context:** PCI-DSS, PIPEDA  
**Framework:** NIST Cybersecurity Framework (CSF) v1.1  
**Document Owner:** Global Administrator  
**Last Reviewed:** May 2026

---

## 1. Overview

This document maps the security controls implemented as part of the FatiuLab Security Privileged Identity & Cloud Security Program to the NIST Cybersecurity Framework (CSF). It demonstrates that the technical controls built across Weeks 1-3 form a coherent, framework-aligned security program — not just a collection of configurations.

**Framework structure:**

```
NIST CSF
├── Identify (ID)     — Know your assets, risks, and governance
├── Protect (PR)      — Implement safeguards to ensure delivery
├── Detect (DE)       — Identify cybersecurity events
├── Respond (RS)      — Take action on detected events
└── Recover (RC)      — Restore capabilities after incidents
```

**Coverage summary:**

| NIST Function | Categories Covered | Controls Mapped |
|---|---|---|
| Identify | ID.AM, ID.GV, ID.RA | 3 |
| Protect | PR.AC, PR.DS, PR.PT | 14 |
| Detect | DE.CM, DE.AE | 8 |
| Respond | RS.CO, RS.AN | 2 |
| Recover | RC.RP | 1 |

---

## 2. Control Mapping — By NIST CSF Function

---

### IDENTIFY (ID)

The Identify function establishes the foundation for an effective cybersecurity program — understanding assets, risks, and governance structures.

---

#### ID.AM — Asset Management

*The data, personnel, devices, systems, and facilities that enable the organization to achieve business purposes are identified and managed.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| ID.AM-1 | Physical devices and systems within the organization are inventoried | Azure resource inventory via Defender for Cloud — 3 resources tracked | P3-DFC-07-Inventory.png |
| ID.AM-2 | Software platforms and applications within the organization are inventoried | Defender for Cloud inventory — storage account, subscription, IAM user | P3-DFC-07-Inventory.png |
| ID.AM-6 | Cybersecurity roles and responsibilities for the entire workforce are established | Users defined with explicit roles — GA, Security Admin, Standard User, Break-glass | P3-BL-03-Users-List.png |

---

#### ID.GV — Governance

*The policies, procedures, and processes to manage and monitor the organization's regulatory, legal, risk, environmental, and operational requirements are understood and inform the management of cybersecurity risk.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| ID.GV-1 | Organizational cybersecurity policy is established and communicated | CIS Azure Foundations Benchmark v2.0.0 assigned as compliance standard | P3-AZP-01-CIS-Policy-Assignment.png |
| ID.GV-3 | Legal and regulatory requirements regarding cybersecurity are understood and managed | PCI-DSS and PIPEDA requirements mapped to all controls in this program | This document |
| ID.GV-4 | Governance and risk management processes address cybersecurity risks | Risk register maintained with 12 identified risks, residual risk tracked | risk-register.md |

---

#### ID.RA — Risk Assessment

*The organization understands the cybersecurity risk to organizational operations, assets, and individuals.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| ID.RA-1 | Asset vulnerabilities are identified and documented | Defender for Cloud CSPM — continuous vulnerability assessment | P3-DFC-08-Secure-Score-Baseline.png, P3-DFC-15-Secure-Score-After.png |
| ID.RA-3 | Threats, both internal and external, are identified and documented | Risk register — 12 risks identified covering external attackers and insider threats | risk-register.md |
| ID.RA-5 | Threats, vulnerabilities, likelihoods, and impacts are used to determine risk | Risk register — likelihood × impact scoring, inherent and residual risk tracked | risk-register.md |

---

### PROTECT (PR)

The Protect function supports the ability to limit or contain the impact of a cybersecurity event.

---

#### PR.AC — Identity Management, Authentication, and Access Control

*Access to physical and logical assets and associated facilities is limited to authorized users, processes, and devices, and is managed consistent with the assessed risk of unauthorized access.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| PR.AC-1 | Identities and credentials are issued, managed, verified, revoked, and audited | Entra ID user lifecycle — 4 users, defined roles, P2 licensed | P3-BL-03-Users-List.png |
| PR.AC-2 | Physical access to assets is managed and protected | N/A — cloud-only environment | — |
| PR.AC-3 | Remote access is managed | CA policies enforce MFA for all remote access — no exception | P3-CA-00-Policy-List.png |
| PR.AC-4 | Access permissions and authorizations are managed incorporating least privilege | PIM JIT access — no standing privilege, eligible assignments only | P3-PIM-03-Eligible-Assignments.png |
| PR.AC-5 | Network integrity is protected | Storage account public network access disabled, network default deny | P3-DFC-11-Remediation-NetworkAccess-Disabled.png |
| PR.AC-6 | Identities are proofed and bound to credentials | Entra ID P2 — identity verification, MFA bound to all accounts | P3-BL-02-P2-License-Assigned.png |
| PR.AC-7 | Users, devices, and other assets are authenticated commensurate with the risk | MFA required for all users (CA01) and all admins (CA02) — risk-based enforcement | P3-CA-01-Require-MFA-All-Users.png, P3-CA-02-Require-MFA-Admins.png |

---

#### PR.AT — Awareness and Training

*The organization's personnel and partners are provided cybersecurity awareness education and are trained to perform their cybersecurity duties and responsibilities.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| PR.AT-2 | Privileged users understand their roles and responsibilities | PIM operating model documented — admin training on JIT workflow | pim-design.md |

---

#### PR.DS — Data Security

*Information and records (data) are managed consistent with the organization's risk strategy to protect the confidentiality, integrity, and availability of information.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| PR.DS-1 | Data-at-rest is protected | Storage account encryption enabled (MMK) — default Azure encryption | P3-DFC-06-Storage-Account-Overview.png |
| PR.DS-2 | Data-in-transit is protected | Secure transfer required on storage account — TLS 1.2 minimum | P3-DFC-04-Storage-Account-Config.png |
| PR.DS-5 | Protections against data leaks are implemented | Shared key access disabled — all access via Entra ID RBAC | P3-DFC-10-Remediation-SharedKey-Disabled.png |

---

#### PR.IP — Information Protection Processes and Procedures

*Security policies, processes, and procedures are maintained and used to manage protection of information systems and assets.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| PR.IP-1 | A baseline configuration of information technology/industrial control systems is created and maintained | CIS benchmark baseline established — 66/81 controls | P3-AZP-05-CIS-Benchmark-Results.png |
| PR.IP-3 | Configuration change control processes are in place | CA policy change detection (DET-04) — all changes alerted and reviewed | P3-DET-04-CA-Policy-Modified.png |

---

#### PR.PT — Protective Technology

*Technical security solutions are managed to ensure the security and resilience of systems and assets, consistent with related policies, procedures, and agreements.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| PR.PT-1 | Audit/log records are determined, documented, implemented, and reviewed | Sentinel workspace with 11 connectors — AuditLogs, SigninLogs, AzureActivity | P3-SEN-03-Data-Connectors.png |
| PR.PT-3 | The principle of least functionality is incorporated | Legacy authentication blocked (CA03) — only modern auth protocols permitted | P3-CA-03-Block-Legacy-Auth.png |

---

### DETECT (DE)

The Detect function enables the timely discovery of cybersecurity events.

---

#### DE.AE — Anomalies and Events

*Anomalous activity is detected and the potential impact of events is understood.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| DE.AE-1 | A baseline of network operations and expected data flows is established | Defender for Cloud Secure Score baseline — 50% pre-remediation | P3-DFC-08-Secure-Score-Baseline.png |
| DE.AE-2 | Detected events are analyzed to understand attack targets and methods | 8 Sentinel analytics rules — MITRE ATT&CK mapped detection | P3-DET-08-Defender-High-Severity.png |
| DE.AE-3 | Event data are collected and correlated from multiple sources | Sentinel aggregating AuditLogs + SigninLogs + AzureActivity | P3-SEN-03-Data-Connectors.png |

---

#### DE.CM — Security Continuous Monitoring

*The information system and assets are monitored to identify cybersecurity events and verify the effectiveness of protective measures.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| DE.CM-1 | The network is monitored to detect potential cybersecurity events | Azure Activity logs streaming to Sentinel — DET-06, DET-08 | P3-SEN-03-Data-Connectors.png |
| DE.CM-3 | Personnel activity is monitored to detect potential cybersecurity events | Entra ID AuditLogs — PIM activations, CA changes, MFA modifications | P3-PIM-07b-Activation-Audit-Log.png |
| DE.CM-6 | External service provider activity is monitored | Defender for Cloud CSPM — continuous third-party resource assessment | P3-DFC-07-Inventory.png |
| DE.CM-7 | Monitoring for unauthorized personnel, connections, devices, and software is performed | DET-05 Impossible Travel, DET-07 Break-glass — behavioral anomaly detection | P3-DET-07-Break-Glass-Sign-In.png |

---

### RESPOND (RS)

The Respond function supports the ability to contain the impact of a potential cybersecurity incident.

---

#### RS.CO — Communications

*Response activities are coordinated with internal and external stakeholders.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| RS.CO-2 | Incidents are reported consistent with established criteria | Security contact email configured — high severity alerts notified | P3-DFC-12-Remediation-SecurityContact.png |
| RS.CO-3 | Information is shared consistent with response plans | Incident tabletop — RACI defines who communicates what | raci.md |

---

#### RS.AN — Analysis

*Analysis is conducted to ensure effective response and support recovery activities.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| RS.AN-1 | Notifications from detection systems are investigated | Sentinel analytics rules — incidents created for all alerts | P3-DET-08-Defender-High-Severity.png |
| RS.AN-3 | Forensics are performed | Entra ID audit logs + PIM audit history — full forensic trail | P3-PIM-07-Audit-Log.png, P3-PIM-07b-Activation-Audit-Log.png |

---

### RECOVER (RC)

The Recover function supports timely recovery to normal operations to reduce the impact from a cybersecurity event.

---

#### RC.RP — Recovery Planning

*Recovery processes and procedures are executed and maintained to ensure restoration of systems or assets affected by cybersecurity incidents.*

| Control ID | Control Description | Implementation | Evidence |
|---|---|---|---|
| RC.RP-1 | Recovery plan is executed during or after a cybersecurity incident | Break-glass account — emergency access procedure defined | pim-design.md |

---

## 3. PCI-DSS Cross-Reference

For a FinTech under PCI-DSS, the following requirement-to-control mapping demonstrates compliance:

| PCI-DSS Requirement | Requirement Description | Controls Implemented |
|---|---|---|
| Req 7.2 | Restrict access to system components | PIM JIT access, CA policies, least privilege |
| Req 7.2.4 | Review user accounts and access privileges quarterly | AR-01 quarterly privileged group review |
| Req 8.2 | User identification and authentication management | Entra ID user lifecycle, unique accounts per user |
| Req 8.3.4 | Account lockout after failed authentication | Entra ID smart lockout (default) |
| Req 8.4 | MFA for all access to cardholder data environment | CA01 MFA all users, CA02 MFA admins |
| Req 8.5 | MFA systems are protected from misuse | PIM MFA on activation, DET-03 MFA disabled detection |
| Req 10.2 | Audit log events are recorded | Sentinel — AuditLogs, SigninLogs, AzureActivity |
| Req 10.3 | Audit logs are protected from destruction | Sentinel workspace — centralized, protected log storage |
| Req 11.3 | External and internal vulnerabilities are identified | Defender for Cloud CSPM continuous assessment |
| Req 12.10 | Incident response plan is in place | Security contact configured, tabletop exercise completed |

---

## 4. Control Coverage Heatmap

```
NIST CSF Function    Coverage
─────────────────────────────────────────────
Identify (ID)        ████████░░  ~70%
Protect (PR)         ██████████  ~90%
Detect (DE)          ████████░░  ~75%
Respond (RS)         ██████░░░░  ~50%
Recover (RC)         ████░░░░░░  ~30%
```

**Strong coverage:** Protect and Detect — where the most lab work was concentrated.

**Gaps — Respond and Recover:** These functions require formal incident response procedures, communication plans, and tested recovery processes. The tabletop exercise in `06-Incident-Tabletop` partially addresses Respond. Recover remains the weakest function — a formal recovery plan is a planned improvement.

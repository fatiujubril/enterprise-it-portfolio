# Cybersecurity Projects

Identity and cloud security engineering built around real failure scenarios — how identity and cloud controls break, how those failures surface in telemetry, and how they are detected, contained, remediated, and governed so they don't recur.

Each project is built end to end in a controlled lab, evidence-backed with screenshots organized by phase, and documented to the standard expected in production: design decisions with reasoning, detection methodology, and honest analysis of what broke and why.

---

## Project 03 — Privileged Identity & Cloud Security Program
**Status: Complete**

Enterprise-grade Privileged Identity & Cloud Security Program built from scratch on Microsoft Entra ID and Azure for a fictional 200-person FinTech regulated under PCI-DSS and PIPEDA. Not a checklist of configurations — a complete security program spanning identity governance, cloud posture, detection engineering, risk management, and a tested incident response capability.

Built across four phases:

- **Identity governance** — 5 Conditional Access policies (MFA enforcement, legacy auth eliminated), PIM with JIT access and approval workflows, 2 automated access reviews, break-glass account with monthly monitoring
- **Cloud security posture** — Defender for Cloud CSPM, storage hardening, CIS Azure Foundations Benchmark v2.0.0, documented exception register with compensating controls
- **Detection engineering** — 8 custom Sentinel analytics rules across AuditLogs, SigninLogs, and AzureActivity, mapped to MITRE ATT&CK, with a full false-positive tuning log
- **GRC & tabletop** — 12-risk register with inherent and residual scoring, NIST CSF control mapping, 7-item exception register, and an AiTM phishing tabletop exercise with RACI and improvement backlog

**Results:** Secure Score 50% → 67% · CIS 66/81 controls (81%) · MCSB 61/63 (97%) · 8 detection rules · 4 MITRE techniques · 12 risks documented · 0 residual risks at High or Critical

**Demonstrates:** Entra ID PIM · Zero Trust · Conditional Access Design · Defender for Cloud · CIS Benchmark · Microsoft Sentinel · KQL Detection Engineering · MITRE ATT&CK · NIST CSF · Risk Register · GRC · Incident Tabletop

📂 [View Project](./project-03-privileged-identity-cloud-security-program/README.md)

---

## Project 02 — Conditional Access Misconfiguration & Identity Attack Simulation
**Status: Complete**

Demonstrates how a legitimate administrative decision made under business pressure creates a high-risk MFA bypass for a high-value executive identity — with no external attacker, no malware, and no phishing involved.

The project builds a secure Conditional Access baseline, introduces a controlled misconfiguration, detects the resulting MFA bypass through Entra ID sign-in telemetry, then performs containment, remediation, and proper MFA re-enrollment — making the case that identity governance failures are frequently more dangerous than external attacks precisely because they look like normal administration.

**Demonstrates:** Entra ID · Conditional Access · MFA Enforcement Validation · Identity Governance · Sign-in Log Analysis · KQL Detection · MITRE ATT&CK · Incident Response Lifecycle

📂 [View Project](./project-02-ca-misconfiguration-identity-attack/README.md)

---

## Project 01 — Identity Compromise via Phishing: M365 Detection, Containment & Hardening
**Status: Complete**

Simulates a credential phishing attack resulting in unauthorized cloud authentication, mailbox-based persistence, and internal phishing propagation — all without malware or endpoint exploitation, which is what makes it representative of how real M365 compromises actually unfold.

Covers the full incident response lifecycle from anomaly detection through containment, remediation, and identity hardening, using Entra ID sign-in logs and Exchange Online telemetry as the primary evidence sources.

**Demonstrates:** Incident Response · Identity Threat Investigation · Entra ID · Exchange Online · MITRE ATT&CK Mapping · KQL · Conditional Access · Evidence Handling

📂 [View Project](./project-01-identity-compromise-m365-phishing/README.md)

---

## Project 04 — KQL Detection Workbook
**Status: In Progress**

Custom KQL detection workbook built in Microsoft Sentinel covering identity, cloud, and endpoint threat scenarios mapped to MITRE ATT&CK.

Includes detection rule engineering, a false positive tuning log, a MITRE coverage matrix, and lessons learned — with emphasis on the tuning work, since a detection that fires constantly is a detection nobody reads.

**Focus areas:** KQL · Microsoft Sentinel · MITRE ATT&CK · Detection Engineering · Threat Hunting · False Positive Tuning

📂 [View Project](./project-04-kql-detection-workbook/README.md)

---

## Planned Projects

Continuing the identity and access theme with a focus on federation and the protocols underneath enterprise SSO — application integration across two identity providers (Okta and Entra ID) using OIDC, SAML, OAuth 2.0, and SCIM, documented at the protocol level rather than the console level.

# Identity & Cloud Security Engineering Portfolio

### Microsoft Entra ID · Conditional Access · PIM · Microsoft Sentinel · Defender XDR · Azure

Hands-on identity and cloud security engineering portfolio built around real-world failure scenarios — how identity and cloud controls break, how failures are detected through telemetry, and how they are remediated, hardened, and governed at program level.

---

## Focus Area

Every project is built to demonstrate the depth, engineering thinking, and documentation quality expected of an **Identity & Cloud Security Engineer** — not just tool familiarity. Each project covers a real failure scenario, detection methodology, remediation, and governance improvement.

---

## Cybersecurity Projects

### Project 01 — Identity Compromise via Phishing: M365 Detection, Containment & Hardening
**Status: Complete**

Simulates a credential phishing attack resulting in unauthorized cloud authentication, mailbox-based persistence, and internal phishing propagation — all without malware or endpoint exploitation.

Covers the full incident response lifecycle from anomaly detection through containment, remediation, and identity hardening using Entra ID sign-in logs and Exchange Online telemetry.

**Demonstrates:** Incident Response · Identity Threat Investigation · Entra ID · Exchange Online · MITRE ATT&CK Mapping · KQL · Conditional Access · Evidence Handling

📂 [View Project](./projects/cybersecurity/project-01-identity-compromise-m365-phishing/README.md)

---

### Project 02 — Conditional Access Misconfiguration & Identity Attack Simulation
**Status: Complete**

Simulates how a legitimate administrative decision made under business pressure creates a high-risk MFA bypass for a high-value executive identity — without any external attacker involved.

Demonstrates detection via sign-in telemetry, containment by removing the CA exception, and recovery through proper MFA re-enrollment — proving that identity governance failures are often more dangerous than external attacks.

**Demonstrates:** Entra ID · Conditional Access · MFA Enforcement Validation · Identity Governance · Sign-in Log Analysis · KQL Detection · MITRE ATT&CK · Incident Response Lifecycle

📂 [View Project](./projects/cybersecurity/project-02-ca-misconfiguration-identity-attack/README.md)

---

### Project 03 — Privileged Identity & Cloud Security Program
**Status: Complete**

Enterprise-grade Privileged Identity & Cloud Security Program built from scratch on Microsoft Entra ID and Azure for a fictional 200-person FinTech regulated under PCI-DSS and PIPEDA.

Delivers a complete security program across four phases: Zero Trust identity governance (Conditional Access, PIM, Access Reviews), cloud security posture management (Defender for Cloud, CIS benchmark), detection engineering (8 custom Sentinel KQL rules mapped to MITRE ATT&CK), and GRC (risk register, NIST CSF control mapping, exception register, incident tabletop).

**Results:** Secure Score 50% → 67% · CIS 66/81 controls (81%) · MCSB 61/63 (97%) · 8 detection rules · 12 risks documented · 0 residual risks at High or Critical

**Demonstrates:** Entra ID PIM · Zero Trust · Conditional Access Design · Defender for Cloud · CIS Benchmark · Microsoft Sentinel · KQL Detection Engineering · MITRE ATT&CK · NIST CSF · Risk Register · GRC · Incident Tabletop

📂 [View Project](./projects/cybersecurity/project-03-privileged-identity-cloud-security-program/README.md)

---

### Project 04 — KQL Detection Workbook
**Status: Not Started**

Custom KQL detection workbook built in Microsoft Sentinel covering identity, cloud, and endpoint threat scenarios mapped to MITRE ATT&CK.

Includes detection rule engineering, false positive tuning log, MITRE coverage matrix, and lessons learned.

**Focus areas:** KQL · Microsoft Sentinel · MITRE ATT&CK · Detection Engineering · Threat Hunting · False Positive Tuning

📂 [View Project](./projects/cybersecurity/project-04-kql-detection-workbook/README.md)

---

## Certifications

Microsoft Certified: Security Operations Analyst (SC-200) · Microsoft 365 Administrator (MS-102) · Azure Fundamentals (AZ-900) · CompTIA Security+ · CSIS · CIOS · Network+ · A+ · ITIL 4 Foundation · ServiceNow Administrator

**In Progress:** Microsoft Identity and Access Administrator (SC-300) · CISSP

---

## About This Portfolio

- Every project is built end-to-end in a controlled lab environment
- Evidence-backed with screenshots organized by phase
- Documented to production standard — executive summaries, MITRE mapping, GRC alignment
- All content is original lab work — no client or confidential data included

---

## Contact

**LinkedIn:** https://www.linkedin.com/in/fatiu-jubril/  
**GitHub:** https://github.com/fatiujubril/enterprise-it-portfolio

Happy to walk through any project, explain design decisions, or discuss how these scenarios map to production environments.

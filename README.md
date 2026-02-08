# Enterprise IT & Cybersecurity Portfolio

Welcome to my **Enterprise IT & Cybersecurity Portfolio**.

This repository showcases hands-on projects across **Cybersecurity**, **Identity & Access Management (IAM)**, and **Enterprise Systems Administration**, built to reflect **real-world enterprise environments**—not toy labs.

Each project focuses on how security and IT systems actually fail, how those failures are detected, and how they are remediated and governed long-term.

---

## Who This Portfolio Is For

This portfolio is designed for reviewers hiring into roles such as:

- **Cybersecurity Engineer / SOC / SecOps**
- **Security Engineer (Microsoft cloud–centric)**
- **Identity & Access Management (IAM / Identity Analyst)**
- **Systems, Network, or Cloud Administrator**
- **Hybrid IT / Security roles**

If you are evaluating hands-on capability, investigative thinking, or operational maturity, this repository is meant to make that visible quickly.

---

## About Me

I am an IT and Cybersecurity professional with hands-on experience supporting **Microsoft-centric enterprise environments**, including:

- Microsoft 365
- Microsoft Entra ID (Azure AD)
- Conditional Access & MFA enforcement
- Identity security and governance
- Incident response and operational security

My work sits at the intersection of **security engineering**, **identity**, and **enterprise IT operations**, with a strong emphasis on documentation, repeatability, and governance.

---

## How This Portfolio Is Structured

Projects are organized into two primary domains:

### 🔐 Cybersecurity Projects
Identity security, detection, response, and governance-focused scenarios.

### 🛠️ Systems & IT Engineering Projects
Enterprise IT operations, administration, automation, and infrastructure  
(**currently in progress**).

Each project contains:
- A dedicated README
- Clear objectives and scope
- Investigation and remediation steps
- Evidence (screenshots, logs, configuration states)
- Lessons learned and governance improvements

---

## 🔐 Cybersecurity Projects

These projects simulate **realistic enterprise threat scenarios**, including both:
- External attack techniques (e.g., phishing)
- Internal control failures (e.g., identity misconfiguration)

The emphasis is not just on fixing issues—but on **how they were detected**, **why they occurred**, and **how to prevent recurrence**.

---

### 🔹 Project 01 – Microsoft 365 Phishing Incident Response

A simulated phishing incident involving a Microsoft 365 user account, focused on **email-based threat detection, investigation, and response**.

This project mirrors how phishing incidents are handled in production Microsoft environments using native tooling and structured response workflows.

**Key stages covered:**
- Phishing detection and triage
- Mail flow and message trace analysis
- Compromised account investigation
- Mailbox rule review and cleanup
- Incident containment and recovery
- Post-incident security hardening

📂 **Project Location:**  
[`projects/project-01-m365-phishing-incident-response`](./projects/cybersecurity/project-01-m365-phishing-incident-response/README.md)

🔎 **Key Focus Areas:**
- Microsoft 365 security operations
- Email threat investigation
- Incident response lifecycle
- Account compromise remediation
- Operational security hygiene

---

### 🔹 Project 02 – Conditional Access Failure & Identity Attack Simulation

An identity security simulation demonstrating how a **Conditional Access misconfiguration** can silently bypass MFA enforcement for a high-value executive account—without phishing, malware, or external attackers.

This project focuses on **identity governance failure**, showing how legitimate administrative actions can unintentionally weaken security controls.

**Incident lifecycle covered:**
- Secure identity baseline establishment
- Conditional Access misconfiguration introduction
- Detection via Entra ID sign-in telemetry
- Investigation and impact analysis
- Containment, remediation, and recovery
- Governance improvements and lessons learned

📂 **Project Location:**  
[`projects/Project-02-Conditional-Access-Failure-Identity-Attack`](./projects/cybersecurity/project-02-Conditional-Access-Failure-Identity-Attack/README.md)

🔎 **Key Focus Areas:**
- Microsoft Entra ID & Conditional Access
- MFA enforcement validation
- Identity misconfiguration detection
- Telemetry-driven investigation
- Security governance and exception handling

---

### 🔹 Project 03 – Detection Engineering & SOC Enhancements (In Progress)

An active, evolving project focused on **detection engineering, alert quality, and SOC efficiency**.

This project will expand into:
- KQL-based detection logic
- Noise reduction and false-positive tuning
- Identity-driven detections
- Investigation and response runbooks
- SOAR-style enrichment and automation concepts

📌 **Status:** Work in progress  
📂 **Location:** To be added under `projects/cybersecurity/`

---

## 🛠️ Systems & IT Engineering Projects (In Progress)

This section will include hands-on enterprise IT projects such as:

- Microsoft 365 administration
- Identity lifecycle management (Joiner / Mover / Leaver)
- Windows Server & Active Directory operations
- Endpoint and device management
- Automation with PowerShell
- Backup, recovery, and operational governance

📌 **Status:** Actively being developed  
Projects will be added incrementally as they are completed.

---

## Resume & Applications

I maintain **multiple tailored PDF resumes** aligned to different role types (e.g., Cybersecurity, IAM, Systems Administration).

Because resumes are customized per role and organization, a single static resume is not linked here.

If you are reviewing this portfolio as part of an application or interview process, I’m happy to:
- Share the resume version aligned to the role
- Walk through any project live
- Explain design decisions, trade-offs, and real-world parallels

---

## How to Review This Portfolio

If you have **1–2 minutes**:
- Open a project README
- Skim the scenario, findings, and remediation

If you have **5–10 minutes**:
- Review the investigation steps
- Look through evidence screenshots/logs
- Read the governance and lessons learned sections

Each project is designed to be explainable end-to-end.

---

## Ongoing Development

This portfolio is actively maintained as new projects are completed and existing ones are refined.

Future work includes:
- Expanded detection engineering
- Identity governance automation
- Systems hardening and operational baselines
- Deeper SOC-style workflows

---

## Contact & Next Steps

If you’re hiring for **Cybersecurity**, **IAM**, or **Enterprise IT** roles in Canada (GTA / Hybrid / Remote), I’m open to conversations.

I’m happy to:
- Walk through any project
- Answer deep technical questions
- Discuss how these scenarios translate to real production environments

---

### Disclaimer

All projects use **lab environments and sanitized data**.  
No client, employer, or confidential information is included.

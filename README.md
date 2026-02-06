# Enterprise IT Portfolio

Welcome to my **Enterprise IT Portfolio**.  
This repository showcases hands-on projects across **Cybersecurity**, **Identity & Access Management (IAM)**, and **Enterprise Systems Administration**.

Each project is designed to reflect **real-world enterprise scenarios**, with an emphasis on:
- Secure-by-design architecture
- Misconfiguration risk and failure modes
- Detection through telemetry and logs
- Incident response and recovery
- Long-term governance and operational controls

All projects are documented end-to-end and structured to be easily navigable for both technical and non-technical reviewers.

---

## About Me

I am an IT and Cybersecurity professional with hands-on experience supporting Microsoft-centric enterprise environments, including **Microsoft 365**, **Microsoft Entra ID (Azure AD)**, **Conditional Access**, and **cloud identity security**.

My core focus areas include:
- Identity & Access Management (IAM)
- Security Operations & Incident Response
- Conditional Access and MFA enforcement
- Enterprise IT operations and governance

This portfolio is intentionally structured to allow reviewers to quickly explore projects relevant to **security-focused**, **systems-focused**, or **hybrid IT roles**.

---

## Portfolio Structure

Projects in this repository are organized into two primary domains:

### 🔐 Cybersecurity Projects
Projects focused on identity security, detection, response, and security governance.

### 🛠️ Systems & IT Engineering Projects
Projects focused on enterprise IT operations, administration, automation, and infrastructure (in progress).

Each project contains its own README and supporting documentation to guide reviewers through the scenario, decisions made, evidence collected, and outcomes.

---

## Cybersecurity Projects

This section contains hands-on security projects focused on **realistic enterprise threat scenarios**, spanning identity compromise, email security, detection, and incident response.

Projects are intentionally designed to demonstrate both **external attack handling** and **internal security control failure**, reflecting how incidents occur in real environments.

---

### 🔹 Project 01 – Microsoft 365 Phishing Incident Response

A simulated phishing incident involving a Microsoft 365 user account, focused on **email-based threat detection and response**.

This project demonstrates how phishing attacks are investigated and remediated in Microsoft-centric environments using native security tooling and operational workflows.

**Key stages covered:**
- Phishing detection and triage
- Mail flow and message trace analysis
- Compromised account investigation
- Mailbox rule review and cleanup
- Incident containment and recovery
- Post-incident hardening actions

📂 **Project Location:**  
[`projects/project-01-m365-phishing-incident-response`](./projects/cybersecurity/project-01-m365-phishing-incident-response/README.md)

🔎 **Key Focus Areas:**
- Microsoft 365 security operations
- Email threat investigation
- Account compromise response
- Incident containment and remediation
- Operational security hygiene

---

### 🔹 Project 02 – Conditional Access Failure & Identity Attack Simulation

A real-world identity security simulation demonstrating how a **Conditional Access misconfiguration** can silently bypass MFA enforcement for a high-value executive account — without phishing, malware, or external attackers.

This project focuses on **internal policy exception abuse** and identity governance failure, highlighting how security controls can be weakened by legitimate administrative actions.

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
- Incident response lifecycle
- Security governance and exception handling

---

## Systems & IT Engineering Projects (Upcoming)

This section will include hands-on projects focused on enterprise IT operations, such as:
- Microsoft 365 administration
- Identity lifecycle management (Joiner / Mover / Leaver)
- Endpoint and device management
- Automation and operational tooling
- Enterprise IT governance and standardization

Projects in this category are actively being developed and will be added incrementally.

---

## How to Use This Portfolio

- **Hiring managers:** Start with each project’s README for a high-level overview.
- **Technical reviewers:** Dive into individual markdown files within each project for detailed implementation and analysis.
- **Security roles:** Focus on detection, response, and governance sections.
- **Systems roles:** Focus on environment setup, operational decisions, and controls.

Each project is designed to be explainable from start to finish and mapped to real enterprise environments.

---

## Contact & Next Steps

This portfolio is actively maintained and expanded as new projects are completed.

If you are reviewing this repository as part of an application or interview process, I am happy to:
- Walk through any project live
- Explain design decisions and trade-offs
- Discuss how these scenarios translate to real-world enterprise environments

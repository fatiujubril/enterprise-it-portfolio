# Project 02 – Conditional Access Failure & Identity Attack Simulation

## Overview

This project simulates a real-world identity security incident caused by a **Conditional Access misconfiguration** in Microsoft Entra ID. It demonstrates how a legitimate administrative decision, made under business pressure, can unintentionally introduce a high-risk authentication bypass for a high-value identity.

The project focuses on **internal Conditional Access exception abuse**, not phishing, malware, or external attackers, reflecting a common but often under-detected identity risk in Microsoft 365 environments.

---

## How to Navigate This Project

This project is structured as a **guided incident lifecycle**.  
Readers are encouraged to follow the sections in order.

### 1️⃣ Environment & Baseline Setup
Establishes the secure starting point of the tenant and identity controls.

- 🔹 [Tenant Overview](environment-setup/tenant-overview.md)
- 🔹 [Conditional Access Baseline](environment-setup/conditional-access-baseline.md)
- 🔹 [Logging Architecture](environment-setup/logging-architecture.md)

---

### 2️⃣ Attack Simulation – Identity Misconfiguration
Documents how a Conditional Access exception introduced an authentication bypass.

- 🔹 [Attack Scenario & Timeline](attack-simulation/attack-scenario.md)
- 🔹 [Misconfiguration Details](attack-simulation/misconfiguration-details.md)

---

### 3️⃣ Detection & Analysis
Shows how the issue was identified using sign-in telemetry and policy evaluation.

- 🔹 [Investigation Notes](detection-analysis/investigation-notes.md)
- 🔹 [Detection Logic](detection-analysis/detection-logic.md)
- 🔹 [KQL Queries](detection-analysis/kql-queries.md)

---

### 4️⃣ Incident Response
Covers containment, remediation, and recovery actions taken to restore security.

- 🔹 [Containment](incident-response/containment.md)
- 🔹 [Remediation](incident-response/remediation.md)
- 🔹 [Recovery](incident-response/recovery.md)

---

### 5️⃣ Lessons Learned & Improvements
Highlights governance gaps and security improvements identified.

- 🔹 [Security Improvements](lessons-learned/security-improvements.md)

---

## Scenario Summary (Quick Read)

An executive user (CEO) experienced an authentication disruption following a mobile device change that affected Microsoft Authenticator. To restore business access quickly, IT administrators temporarily excluded the CEO account from the Conditional Access policy enforcing MFA.

The exclusion lacked expiration or review controls. As a result, the CEO account silently authenticated using **single-factor authentication**, bypassing MFA enforcement until detected through sign-in telemetry.

---

## Environment Summary

- **Tenant Type:** Microsoft 365 Business (dedicated lab tenant)
- **Identity Provider:** Microsoft Entra ID (cloud-only)
- **Licensing:**
  - Microsoft 365 Business Standard (trial)
  - Microsoft Entra ID P2 (trial)
- **Scope:** Identity and Conditional Access only

No endpoint compromise, email phishing, or SIEM automation was used.

---

## Key Takeaways

- Conditional Access policies can appear secure while failing to apply
- MFA configured ≠ MFA enforced
- High-value identities should never rely on ad-hoc policy exceptions
- Sign-in telemetry is the authoritative source for identity security validation
- Governance failures can create attack paths without malicious intent

---

## Skills Demonstrated

- Microsoft Entra ID & Conditional Access
- Identity threat modeling
- Sign-in log analysis
- MFA enforcement validation
- Incident response lifecycle (Detection → Containment → Recovery)
- Security governance and exception handling
- Technical documentation and evidence handling

---

## Conclusion

This project demonstrates how identity security failures can occur through internal misconfiguration rather than external attack, and how disciplined detection and response processes can effectively identify and remediate such risks.

It reinforces the importance of **identity governance**, **telemetry-driven validation**, and **secure exception handling** in modern cloud environments.

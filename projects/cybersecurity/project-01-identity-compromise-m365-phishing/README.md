# Project 01 - Identity Compromise via Phishing: M365 Detection, Containment & Hardening

> **Type:** Incident Response Simulation
> **Environment:** Microsoft 365 / Entra ID / Exchange Online
> **Difficulty:** Intermediate
> **Status:** Complete

---

## Executive Summary

A standard user account was compromised via credential phishing, resulting in unauthorized cloud authentication, mailbox-based persistence, and internal phishing propagation - all without malware or endpoint exploitation.

This project documents the **full incident response lifecycle**: from anomaly detection through containment, remediation, and identity hardening. It demonstrates how an identity-first attacker abuses cloud-native features to persist undetected, and how a security engineer detects, investigates, and closes the gap.

**Key Outcome:** Attacker access fully contained and revoked. MFA enforced via Conditional Access post-incident. Malicious inbox rule removed. Identity posture hardened.

---

## Threat Scenario

| Field | Detail |
|---|---|
| **Incident Type** | Account Compromise - Phishing |
| **Attack Vector** | Credential theft via phishing link |
| **Affected Asset** | Exchange Online mailbox (standard user) |
| **Persistence Mechanism** | Malicious inbox rule suppressing security alerts |
| **Impact** | Outbound phishing sent from trusted internal identity |
| **Detection Sources** | Entra ID sign-in logs, Exchange Online message trace |
| **Resolution** | Contained and remediated |

**No malware. No endpoint compromise. No privilege escalation.**
This is a pure identity attack - the most common and underestimated threat vector in Microsoft 365 environments.

---

## Environment

| Component | Detail |
|---|---|
| **Identity Platform** | Microsoft Entra ID |
| **Email Platform** | Exchange Online |
| **Endpoint** | Windows 11 (domain-joined) |
| **User Privilege** | Standard (non-administrative) |
| **MFA Status** | Registered - not enforced at time of compromise |
| **Conditional Access** | Not applied to Outlook Web at baseline |

---

## Attack Chain & MITRE ATT&CK Mapping

```
[T1566.002] Phishing Link
      |
      v
[T1078] Valid Account Authentication (Single-Factor, Foreign IP)
      |
      v
[T1114.003] Mailbox Rule Creation - Security Alert Suppression
      |
      v
[T1534] Internal Spearphishing from Trusted Identity
```

| Technique | ID | Description |
|---|---|---|
| Phishing: Spearphishing Link | T1566.002 | Credential harvesting via phishing |
| Valid Accounts | T1078 | Authenticated using stolen credentials, no malware |
| Email Collection: Inbox Rule Manipulation | T1114.003 | Rule created to suppress security notifications |
| Internal Spearphishing | T1534 | Compromised account used to phish internal users |

---

## Incident Timeline

### Phase 1 - Initial Access: Credential Abuse

Attacker authenticated to Outlook Web App using phished credentials from a foreign IP address.

**Indicators:**
- New geographic location and IP address
- Browser-based OWA authentication
- Single-factor authentication success (MFA not enforced)

Evidence: [06-entra-suspicious-signin.png](Evidence/screenshots/06-entra-suspicious-signin.png)

---

### Phase 2 - Persistence: Malicious Inbox Rule

Attacker created an inbox rule to suppress security-related email notifications.

**Rule logic:**
- Trigger: Subject or body contains "security"
- Action: Move to Archive + Mark as Read

Evidence: [07-mailbox-rule-created.png](Evidence/screenshots/07-mailbox-rule-created.png)

---

### Phase 3 - Impact: Outbound Phishing

The compromised identity was used to send phishing emails to controlled recipients.

Evidence: [08-outbound-phishing-message-trace.png](Evidence/screenshots/08-outbound-phishing-message-trace.png)

---

### Phase 4 - Detection & Investigation

Cross-referencing Entra ID sign-in logs with Exchange Online telemetry reconstructed a coherent incident timeline.

**Confirmed findings:**
- Unauthorized sign-ins aligned with inbox rule creation timestamp
- No lateral movement, OAuth abuse, email forwarding, or additional compromised accounts detected

Evidence: [09-signin-log-analysis.png](Evidence/screenshots/09-signin-log-analysis.png), [10-mailbox-rule-investigation.png](Evidence/screenshots/10-mailbox-rule-investigation.png)

---

### Phase 5 - Containment & Remediation

Actions executed in deliberate sequence to terminate access without destroying investigative evidence:

1. **Session revocation** - invalidated all active attacker tokens
2. **Password reset** - forced change on next login
3. **Inbox rule removal** - eliminated persistence mechanism
4. **MFA enforcement** - applied via Conditional Access policy post-incident

Evidence: [11-session-revocation.png](Evidence/screenshots/11-session-revocation.png), [12-password-reset.png](Evidence/screenshots/12-password-reset.png), [13-mailbox-rule-removed.png](Evidence/screenshots/13-mailbox-rule-removed.png), [14-mfa-enabled.png](Evidence/screenshots/14-mfa-enabled.png)

---

## Root Cause Analysis

| Factor | Detail |
|---|---|
| **Primary Cause** | Credentials compromised via phishing; MFA not enforced |
| **Contributing Factor 1** | MFA registered but not required via Conditional Access |
| **Contributing Factor 2** | No Conditional Access policy scoped to Outlook Web |
| **Contributing Factor 3** | No alerting configured on inbox rule creation |

---

## Security Improvements & Recommendations

| Control | Priority | Implementation |
|---|---|---|
| Enforce MFA via Conditional Access for all cloud apps | Critical | Entra ID > Conditional Access > New Policy |
| Block legacy authentication protocols | Critical | Conditional Access > Client Apps condition |
| Alert on inbox rule creation and modification | High | Microsoft Sentinel analytics rule |
| Sign-in risk-based Conditional Access | High | Entra ID Protection > Risk Policy |
| Periodic mailbox rule audit | Medium | PowerShell or Defender for Cloud Apps |

---

## Analyst Decision Points

- **Prioritized identity telemetry over endpoint forensics** - no endpoint alerts; Entra ID logs were the primary signal
- **Treated inbox rule as persistence, not misconfiguration** - timing and rule logic inconsistent with normal user behavior
- **Used message trace to confirm business impact** before containment - preserving evidence sequence
- **Revoked sessions before password reset** - prevented token reuse during the remediation window
- **Documented negative findings** - confirmed blast radius was limited

---

## Evidence Index

| Phase | Screenshot | Description |
|---|---|---|
| Baseline | 01-ad-user-group-membership.png | Standard user confirmed |
| Baseline | 02-endpoint-identity-network.png | Endpoint identity and network baseline |
| Baseline | 03-endpoint-privilege-groups.png | No local admin privileges |
| Baseline | 04-mailbox-rules-baseline.png | No inbox rules at baseline |
| Baseline | 05-mailbox-forwarding-baseline.png | Email forwarding disabled |
| Initial Access | 06-entra-suspicious-signin.png | Anomalous sign-in - foreign IP, single-factor |
| Persistence | 07-mailbox-rule-created.png | Malicious inbox rule creation |
| Impact | 08-outbound-phishing-message-trace.png | Outbound phishing confirmed |
| Investigation | 09-signin-log-analysis.png | Sign-in log correlation |
| Investigation | 10-mailbox-rule-investigation.png | Inbox rule investigation |
| Containment | 11-session-revocation.png | Active session revocation |
| Containment | 12-password-reset.png | Forced password reset |
| Containment | 13-mailbox-rule-removed.png | Malicious rule removed |
| Hardening | 14-mfa-enabled.png | MFA enforced via Conditional Access |

---

## Skills Demonstrated

Microsoft Entra ID, Exchange Online, Incident Response, Identity Threat Investigation, MITRE ATT&CK Mapping, Log Correlation, Evidence Handling, IR Sequencing, Conditional Access, Security Hardening

---

## How to Review This Project

- Start with the **Executive Summary** for the full picture in 60 seconds
- Follow the **phase-based timeline** for the investigation narrative
- Use the **Evidence Index** to correlate each finding with its screenshot
- Review **Analyst Decision Points** to understand the reasoning, not just the steps

*All activity performed in a controlled lab tenant. No real user data or client information included.*

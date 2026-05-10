**🔐 Microsoft 365 Phishing Incident Response Simulation**
## Project Overview
This project simulates a realistic Microsoft 365 account compromise via phishing, focusing on identity abuse, mailbox persistence, and incident response rather than malware or endpoint exploitation.

## Threat Narrative
This incident simulates a realistic Microsoft 365 phishing-based account compromise affecting a standard user account.

The attacker successfully obtained valid credentials via phishing and authenticated to the account from an unfamiliar location. No malware was deployed; instead, the attacker relied on legitimate cloud authentication and Exchange Online features to persist access.

- **Post-compromise activity focused on:**
  - Establishing mailbox-based persistence
  - Suppressing security notifications
  - Leveraging a trusted internal identity to send outbound phishing emails

This scenario reflects common real-world identity attacks where cloud-native abuse replaces traditional endpoint exploitation.

- **The objective was to demonstrate how a SOC analyst would:**
  - Identify a compromised account
  - Investigate attacker activity
  - Assess impact
  - Contain and remediate the incident
  - Extract lessons learned

All activity was performed in a **controlled lab tenant** using standard Microsoft 365 tooling.

## How to Review This Case Study

This case study is structured to mirror how a SOC analyst or security engineer would investigate and document a real Microsoft 365 phishing incident.

Reviewers are encouraged to:
- Start with the **Incident Summary** for a high-level overview
- Follow the **phase-based narrative** (Initial Access → Persistence → Impact → Investigation → Containment)
- Use the **Evidence Index** to correlate investigative steps with supporting screenshots
- Focus on **analysis and decision-making**, not just screenshots

The emphasis is on realistic attack paths, investigation logic, and correct incident response sequencing.

## Incident Summary (Executive View)
- **Incident Type:** Account Compromise (Phishing)  
- **Attack Vector:** Credential theft via phishing  
- **Affected Asset:** Exchange Online mailbox  
- **User Privilege Level:** Standard user  
- **Persistence Mechanism:** Malicious inbox rule  
- **Impact:** Outbound phishing emails sent from trusted account  
- **Detection Sources:**  
  - Entra ID sign-in logs  
  - Exchange Online message trace  
- **Status:** Contained and remediated 

## Environment Overview
- **Identity Platform:** Microsoft Entra ID
- **Email Platform:** Exchange Online
- **Endpoint:** Windows 11 (domain-joined)
- **User Role:** Non-administrative user
- **Security Controls:**
  - MFA registered (not enforced at time of compromise)
  - No Conditional Access restrictions initially applied

## Baseline State (Pre-Incident)
Before simulating the attack, the environment was validated to ensure a clean baseline:
- User was a **standard domain user**
- No local or domain administrative privileges
- No mailbox rules configured
- Email forwarding disabled
- No abnormal sign-in activity observed
- Security logs reviewed under administrative context
  
This baseline ensured that any subsequent activity could be confidently attributed to the simulated compromise.

## Attack Simulation Overview
The attack was intentionally scoped to credential abuse only, reflecting one of the most common real-world Microsoft 365 incidents.

**Simulated Attacker Capabilities**
- Valid username and password obtained via phishing
- No malware
- No endpoint compromise
- No privilege escalation

## Phase 1 – Initial Access (Credential Abuse)
The attacker authenticated to Outlook on the web using stolen credentials from a **foreign IP address**, creating a clear anomaly in Entra ID sign-in logs.

**Key indicators observed:**
- New geographic location
- New IP address
- Browser-based sign-in
- Successful authentication using single-factor authentication
  
This activity was consistent with credential compromise rather than user error.

### Evidence: Anomalous Sign-In Activity

The sign-in log below shows a successful authentication from an unexpected geographic location using single-factor authentication, consistent with credential compromise.

![Anomalous Entra ID sign-in](evidence/06-entra-suspicious-signin.png)

## Phase 2 – Persistence via Mailbox Rule
After gaining access, the attacker created a malicious inbox rule designed to suppress security-related emails.

**Rule behavior:**
  - Condition: Subject or body contains the keyword "security"
  - **Action:**
    - Move message to Archive
    - Mark message as read
      
This technique delays detection by hiding warning emails while maintaining mailbox functionality.

### Evidence: Mailbox Rule Persistence

The attacker established persistence by creating a malicious inbox rule designed to suppress security-related emails.

![Mailbox rule persistence](evidence/07-mailbox-rule-created.png)

## Phase 3 – Impact (Outbound Phishing)
The compromised mailbox was used to send a small number of phishing-style emails to controlled recipients.

**Observed impact:**
- Emails successfully delivered
- Message trace confirmed sender, recipient, timestamp, and delivery status
- Trust abuse demonstrated without large-scale spamming
  
This escalated the incident from a simple account anomaly to an active security incident.

### Evidence: Outbound Phishing Activity

Exchange Online message trace confirms outbound phishing emails sent from the compromised mailbox.

![Outbound phishing message trace](evidence/08-outbound-phishing-message-trace.png)

## Phase 4 – Investigation & Correlation
During investigation, the following were confirmed:
- Unauthorized sign-ins aligned with mailbox rule creation
- Inbox rule explained lack of user awareness
- Outbound phishing activity confirmed via message trace
- **No evidence of:**
  - Lateral movement
  - OAuth abuse
  - Email forwarding
  - Additional compromised accounts
    
A clear and coherent incident timeline was reconstructed.

## Phase 5 – Containment & Remediation
Containment actions followed correct incident response sequencing:
- Session revocation to invalidate active attacker tokens
- Password reset with forced change on next login
- Removal of malicious inbox rule
- MFA enforcement confirmed post-incident
- User notification (conceptual)
  
This order ensured access was terminated without destroying investigative evidence.

## Analyst Decision Points
Key investigative decisions during this incident included:

- Prioritizing identity and mailbox telemetry over endpoint malware analysis due to the absence of endpoint alerts
- Treating inbox rules as a persistence mechanism rather than a user misconfiguration
- Using message trace to confirm business impact before remediation
- Revoking sessions prior to password reset to prevent token reuse
- Removing malicious mailbox rules before re-enabling user access
  
These decisions mirror real SOC workflows where timing and sequencing directly affect containment success.

## MITRE ATT&CK Mapping
This incident aligns with the following MITRE ATT&CK techniques:

- **T1566.002 – Phishing: Spearphishing Link**  
  Used to capture valid Microsoft 365 credentials.
- **T1078 – Valid Accounts**  
  Attacker authenticated using legitimate credentials without malware.
- **T1114.003 – Email Collection: Inbox Rule Manipulation**  
  Mailbox rules were used to suppress security notifications and maintain persistence.
- **T1534 – Internal Spearphishing**  
  Compromised mailbox used to send phishing emails to other users.

This attack path reflects common identity-based intrusions where attackers exploit cloud-native features instead of deploying malicious payloads.

## Root Cause Analysis
**Primary Cause**
- User credentials compromised via phishing
- Authentication succeeded using single-factor authentication

**Contributing Factors**
- MFA was registered but not enforced
- Conditional Access policies were not applied to Outlook Web access
- No alerting on inbox rule creation

## Lessons Learned
- Credential-only attacks can fully compromise cloud mailboxes
- Inbox rules remain a low-noise persistence mechanism
- MFA registration alone does not prevent compromise
- Identity telemetry is often the earliest and most reliable detection source
- Even a single phishing email constitutes real business risk

## Recommendations
- Enforce MFA via Conditional Access for all cloud applications
- Implement sign-in risk or location-based access controls
- Alert on inbox rule creation and modification
- Alert on first outbound phishing email from a user
- Perform periodic mailbox rule audits

> Screenshots are intentionally limited and selected to demonstrate key decision points rather than exhaustively document every action.

## Evidence Index

| Phase           | Evidence |
|-----------------|----------|
| Baseline        | [01](evidence/01-ad-user-group-membership.png), [02](evidence/02-endpoint-identity-network.png), [03](evidence/03-endpoint-privilege-groups.png), [04](evidence/04-mailbox-rules-baseline.png), [05](evidence/05-mailbox-forwarding-baseline.png) |
| Initial Access  | [06](evidence/06-entra-suspicious-signin.png) |
| Persistence     | [07](evidence/07-mailbox-rule-created.png) |
| Impact          | [08](evidence/08-outbound-phishing-message-trace.png) |
| Investigation   | [09](evidence/09-signin-log-analysis.png), [10](evidence/10-mailbox-rule-investigation.png) |
| Containment     | [11](evidence/11-session-revocation.png), [12](evidence/12-password-reset.png), [13](evidence/13-mailbox-rule-removed.png) |
| Hardening       | [14](evidence/14-mfa-enabled.png) |

## Evidence Handling
All screenshots and supporting artifacts are stored in the `/evidence` directory and referenced by phase and sequence number.

Evidence is intentionally separated from analysis to preserve readability while maintaining traceability between investigative steps and supporting data.

## Skills Demonstrated
- Microsoft Entra ID investigation
- Exchange Online incident response
- Identity-centric threat analysis
- SOC workflow and IR sequencing
- Evidence handling and documentation
- Realistic attacker modeling

## Final Notes
This project intentionally avoids exaggerated techniques or unrealistic tooling.
Every step reflects common, real-world Microsoft 365 phishing incidents encountered by SOC and IR teams.

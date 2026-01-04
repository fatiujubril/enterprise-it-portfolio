🔐 Microsoft 365 Phishing Incident Response Simulation
## Project Overview
This project simulates a realistic Microsoft 365 account compromise via phishing, focusing on identity abuse, mailbox persistence, and incident response rather than malware or endpoint exploitation.

The objective was to demonstrate how a SOC analyst would:

- Identify a compromised account
- Investigate attacker activity
- Assess impact
- Contain and remediate the incident
- Extract lessons learned

All activity was performed in a **controlled lab tenant** using standard Microsoft 365 tooling.

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

## Phase 2 – Persistence via Mailbox Rule
After gaining access, the attacker created a malicious inbox rule designed to suppress security-related emails.

**Rule behavior:**
  - Condition: Subject or body contains the keyword "security"
  - **Action:**
    - Move message to Archive
    - Mark message as read
This technique delays detection by hiding warning emails while maintaining mailbox functionality.

## Phase 3 – Impact (Outbound Phishing)
The compromised mailbox was used to send a small number of phishing-style emails to controlled recipients.

**Observed impact:**
- Emails successfully delivered
- Message trace confirmed sender, recipient, timestamp, and delivery status
- Trust abuse demonstrated without large-scale spamming
This escalated the incident from a simple account anomaly to an active security incident.

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

## Evidence Index
| Phase            | Evidence |
|------------------|----------|
| Baseline         | Screenshots 01–05 |
| Initial Access   | Screenshot 06 |
| Persistence      | Screenshot 07 |
| Impact           | Screenshot 08 |
| Investigation    | Screenshots 09–10 |
| Containment      | Screenshots 11–13 |
| Hardening        | Screenshot 14 |

---

## Skills Demonstrated

- Microsoft Entra ID investigation
- Exchange Online incident response
- Identity-centric threat analysis
- SOC workflow and IR sequencing
- Evidence handling and documentation
- Realistic attacker modeling


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

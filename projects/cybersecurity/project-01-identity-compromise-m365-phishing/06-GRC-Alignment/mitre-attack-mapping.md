# MITRE ATT&CK Mapping

## Techniques Covered

| Phase | Technique ID | Technique Name | Implementation | Detection Method |
|---|---|---|---|---|
| Initial Access | T1566.002 | Phishing: Spearphishing Link | Credential harvesting link sent via email | Email security filtering, user awareness |
| Defense Evasion / Persistence | T1078 | Valid Accounts | Attacker authenticated using stolen credentials | Entra ID sign-in anomaly detection, MFA enforcement |
| Persistence | T1114.003 | Email Collection: Inbox Rule Manipulation | Malicious inbox rule suppressing security alerts | OfficeActivity audit log, Sentinel alert on New-InboxRule |
| Impact | T1534 | Internal Spearphishing | Compromised mailbox used to send internal phishing | Defender for Office 365, message trace anomaly |

## ATT&CK Navigator Coverage

Tactics covered in this simulation:
- Initial Access
- Persistence
- Defense Evasion
- Collection
- Impact

## Detection Coverage Assessment

| Technique | Detected in Sim? | Detection Method Used | Automated Alert? |
|---|---|---|---|
| T1566.002 | No - precondition | N/A - phishing assumed successful | No |
| T1078 | Yes - manual review | Entra ID sign-in log review | No |
| T1114.003 | Yes - manual review | Exchange mailbox audit log | No |
| T1534 | Yes - manual review | Exchange message trace | No |

## Key Gap

No automated detection fired during this incident. All detection was manual.
Implementing Sentinel analytics rules mapped to T1078 and T1114.003 would
convert this from reactive investigation to proactive alerting.

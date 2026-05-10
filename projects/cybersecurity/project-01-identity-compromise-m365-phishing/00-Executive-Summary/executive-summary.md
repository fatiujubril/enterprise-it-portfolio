# Executive Summary

## Incident Overview

| Field | Detail |
|---|---|
| **Incident Type** | Account Compromise - Phishing |
| **Attack Vector** | Credential theft via phishing link |
| **Affected Asset** | Exchange Online mailbox (standard user) |
| **Persistence Mechanism** | Malicious inbox rule suppressing security alerts |
| **Impact** | Outbound phishing sent from trusted internal identity |
| **Detection Sources** | Entra ID sign-in logs, Exchange Online message trace |
| **Resolution** | Contained and remediated |

## Summary

A standard user account was compromised via credential phishing, resulting in unauthorized
cloud authentication, mailbox-based persistence, and internal phishing propagation -
all without malware or endpoint exploitation.

The attacker authenticated using stolen credentials from a foreign IP address, created a
malicious inbox rule to suppress security alerts, then leveraged the trusted identity to
send outbound phishing emails to internal recipients.

## Key Findings

- MFA was registered but not enforced via Conditional Access - this was the primary control gap
- Inbox rules were used as a low-noise persistence mechanism, delaying user awareness
- Identity telemetry (Entra ID sign-in logs) was the earliest and most reliable detection source
- No lateral movement, privilege escalation, or additional account compromise was detected

## Outcome

Attacker access fully contained and revoked. Malicious inbox rule removed. MFA enforced
via Conditional Access post-incident. Identity posture hardened against recurrence.

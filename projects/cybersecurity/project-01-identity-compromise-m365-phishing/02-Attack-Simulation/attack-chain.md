# Attack Chain

## Full Attack Sequence

```
[T1566.002] Phase 1 - Initial Access
Phishing email delivers credential harvesting link
Victim submits Microsoft 365 credentials to attacker-controlled page
      |
      v
[T1078] Phase 2 - Valid Account Authentication
Attacker authenticates to Outlook Web App (OWA)
Single-factor authentication succeeds (MFA not enforced)
Foreign IP address - no location-based block in place
      |
      v
[T1114.003] Phase 3 - Persistence via Inbox Rule
Attacker creates malicious inbox rule:
  - Trigger: subject/body contains "security"
  - Action: move to Archive + mark as read
Suppresses security alert emails from reaching victim
      |
      v
[T1534] Phase 4 - Impact via Internal Spearphishing
Attacker sends phishing emails from compromised trusted account
Recipients more likely to trust email from known internal sender
```

## MITRE ATT&CK Technique Details

| Phase | Technique ID | Technique Name | Detail |
|---|---|---|---|
| Initial Access | T1566.002 | Phishing: Spearphishing Link | Credential harvesting via phishing link |
| Execution | T1078 | Valid Accounts | OWA login using stolen credentials |
| Persistence | T1114.003 | Email Collection: Inbox Rule Manipulation | Rule suppresses security notifications |
| Impact | T1534 | Internal Spearphishing | Trusted identity abused for further phishing |

## Key Observations

- No malware at any stage - entirely credential and cloud-native
- Attack chain completed using only a browser and OWA
- Each phase builds on the previous - credential theft enables persistence, persistence enables impact
- Inbox rule creation is the pivot point between access and impact

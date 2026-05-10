# Detection Logic

## Primary Detection Sources

| Source | Signal | Technique Detected |
|---|---|---|
| Entra ID Sign-in Logs | Foreign IP, single-factor auth, new location | T1078 Valid Accounts |
| Exchange Mailbox Audit | Inbox rule created post-compromise | T1114.003 Inbox Rule Manipulation |
| Exchange Message Trace | Outbound emails to external recipients | T1534 Internal Spearphishing |

## Detection Sequence

1. **Anomalous sign-in detected** - foreign IP address and geography not seen before
2. **MFA gap confirmed** - authentication succeeded on single factor only
3. **Inbox rule correlated** - rule creation timestamp aligns with attacker session
4. **Outbound phishing confirmed** - message trace shows delivery from compromised mailbox

## Detection Gaps Identified

| Gap | Risk | Recommended Control |
|---|---|---|
| No alert on inbox rule creation | High - attacker persistence undetected | Sentinel analytics rule on New-InboxRule |
| No sign-in risk alert | High - foreign IP went undetected | Entra ID Protection risk policy |
| No MFA enforcement | Critical - single-factor auth succeeded | Conditional Access MFA policy |
| No legacy auth block | Medium - additional attack surface | Conditional Access client apps condition |

## Detection Verdict

This attack would not have triggered any automated alerts in the baseline environment.
Detection relied entirely on manual log review, highlighting the critical importance of:
- Enforcing MFA via Conditional Access (not just registering it)
- Configuring alerts on high-risk mailbox operations
- Implementing sign-in risk policies via Entra ID Protection

# Root Cause Analysis

## Primary Cause

User credentials were compromised via phishing and authentication succeeded
using single-factor authentication. MFA was registered but not enforced via
Conditional Access policy, creating a critical control gap.

## Contributing Factors

| Factor | Detail | Impact |
|---|---|---|
| MFA not enforced | MFA was registered but Conditional Access did not require it | Allowed single-factor authentication to succeed |
| No Conditional Access on OWA | Outlook Web access had no location or risk-based restrictions | Foreign IP authentication went unchallenged |
| No inbox rule alerting | No alert configured for inbox rule creation events | Persistence mechanism went undetected |
| No sign-in risk policy | Entra ID Protection not configured to block risky sign-ins | Foreign IP sign-in not challenged automatically |

## Root Cause Chain

```
User clicks phishing link
      |
      v
Credentials submitted to attacker
      |
      v
Attacker authenticates via OWA [MFA not enforced - CONTROL GAP]
      |
      v
Inbox rule created [No alert on rule creation - DETECTION GAP]
      |
      v
Outbound phishing sent [Trust abuse enabled by compromised identity]
```

## Key Lesson

MFA registration is not the same as MFA enforcement.
Organizations frequently register MFA for users but fail to enforce it via
Conditional Access, leaving a window for attackers to authenticate without
completing the MFA challenge. This is one of the most common control gaps
in Microsoft 365 environments.

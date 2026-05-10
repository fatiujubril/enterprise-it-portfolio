# Logging Architecture

## Logs Enabled for This Simulation

| Log Source | Platform | Purpose |
|---|---|---|
| Sign-in Logs | Microsoft Entra ID | Detect anomalous authentication events |
| Audit Logs | Microsoft Entra ID | Track identity and directory changes |
| Mailbox Audit Logs | Exchange Online | Track mailbox rule creation and modification |
| Message Trace | Exchange Online | Confirm outbound email activity |
| Unified Audit Log | Microsoft 365 | Cross-platform activity correlation |

## Key Detection Signals

- **Entra ID sign-in logs** - primary signal for anomalous authentication (foreign IP, single-factor)
- **Exchange mailbox audit** - captures inbox rule creation with timestamps for correlation
- **Message trace** - confirms outbound phishing delivery and recipient scope

## Log Retention

- Entra ID sign-in logs: 30 days (Microsoft 365 Business Premium)
- Unified Audit Log: 90 days default

## Notes

Defender for Office 365 and Microsoft Sentinel were not used in this simulation.
Detection was performed manually using native Microsoft 365 admin portal tooling
to demonstrate baseline SOC investigation skills without reliance on SIEM automation.

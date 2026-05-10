# Evidence Index

All screenshots are stored in the `Evidence/screenshots/` directory.
Each file is numbered sequentially and corresponds to a specific investigation phase.

## Screenshots

| # | Filename | Phase | Description |
|---|---|---|---|
| 01 | 01-ad-user-group-membership.png | Baseline | AD group membership - standard user confirmed, no admin groups |
| 02 | 02-endpoint-identity-network.png | Baseline | Endpoint identity and network configuration at baseline |
| 03 | 03-endpoint-privilege-groups.png | Baseline | Local privilege groups - no local admin confirmed |
| 04 | 04-mailbox-rules-baseline.png | Baseline | Mailbox rules - none configured at baseline |
| 05 | 05-mailbox-forwarding-baseline.png | Baseline | Email forwarding - disabled at baseline |
| 06 | 06-entra-suspicious-signin.png | Initial Access | Anomalous sign-in from foreign IP using single-factor auth |
| 07 | 07-mailbox-rule-created.png | Persistence | Malicious inbox rule created post-compromise |
| 08 | 08-outbound-phishing-message-trace.png | Impact | Outbound phishing emails confirmed via message trace |
| 09 | 09-signin-log-analysis.png | Investigation | Sign-in log correlation and timeline reconstruction |
| 10 | 10-mailbox-rule-investigation.png | Investigation | Inbox rule detail - condition and action reviewed |
| 11 | 11-session-revocation.png | Containment | Active sessions revoked via Entra ID |
| 12 | 12-password-reset.png | Containment | Forced password reset applied |
| 13 | 13-mailbox-rule-removed.png | Remediation | Malicious inbox rule removed |
| 14 | 14-mfa-enabled.png | Hardening | MFA enforced via Conditional Access policy |

## Evidence Handling Notes

- All screenshots captured from controlled lab tenant
- No real user data or client information included
- Files numbered sequentially to support phase-based narrative
- Screenshots selected to demonstrate key decision points, not exhaustive documentation

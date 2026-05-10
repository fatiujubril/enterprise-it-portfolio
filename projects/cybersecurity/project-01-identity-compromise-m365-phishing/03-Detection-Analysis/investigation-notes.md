# Investigation Notes

## Investigation Approach

Investigation prioritized identity and mailbox telemetry over endpoint forensics.
No endpoint alerts were generated, making Entra ID sign-in logs the primary signal.

## Phase-by-Phase Findings

### Phase 1 - Anomalous Sign-In Review

Reviewed Entra ID sign-in logs for the affected user account.

Findings:
- Successful authentication from unrecognized geographic location
- IP address not previously associated with this user
- Authentication method: single-factor (password only)
- Client: browser-based OWA session
- No MFA challenge issued - policy gap confirmed

Evidence: [06-entra-suspicious-signin.png](../Evidence/screenshots/06-entra-suspicious-signin.png)

### Phase 2 - Mailbox Rule Investigation

Reviewed Exchange Online mailbox audit logs for rule changes post-compromise.

Findings:
- New inbox rule created within minutes of anomalous sign-in
- Rule name designed to appear benign
- Rule logic: suppress emails containing "security" keyword
- Action: archive and mark as read - designed to prevent user awareness

Evidence: [07-mailbox-rule-created.png](../Evidence/screenshots/07-mailbox-rule-created.png)
Evidence: [10-mailbox-rule-investigation.png](../Evidence/screenshots/10-mailbox-rule-investigation.png)

### Phase 3 - Outbound Email Review

Ran Exchange Online message trace for the affected mailbox during the compromise window.

Findings:
- Outbound phishing-style emails sent to controlled recipients
- Delivery confirmed - trust abuse demonstrated
- Volume limited - consistent with targeted phishing, not spam campaign

Evidence: [08-outbound-phishing-message-trace.png](../Evidence/screenshots/08-outbound-phishing-message-trace.png)

### Phase 4 - Scope Assessment (Negative Findings)

Investigated for indicators of broader compromise.

Confirmed NOT present:
- No lateral movement to other accounts
- No OAuth application consent grants
- No email forwarding rules configured
- No additional compromised accounts identified
- No privilege escalation attempts

Evidence: [09-signin-log-analysis.png](../Evidence/screenshots/09-signin-log-analysis.png)

## Investigation Conclusion

Single account compromise. Blast radius confirmed as limited to one mailbox.
No evidence of broader infrastructure compromise.

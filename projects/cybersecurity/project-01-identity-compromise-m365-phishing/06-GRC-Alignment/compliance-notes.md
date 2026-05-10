# Compliance Notes

## Relevant Framework Mappings

### NIST SP 800-61 (Incident Response)

| NIST Phase | Project Coverage |
|---|---|
| Preparation | Baseline state documented, logging architecture defined |
| Detection & Analysis | Sign-in log review, mailbox audit correlation, scope assessment |
| Containment | Session revocation, account disable, sequencing rationale documented |
| Eradication | Password reset, malicious rule removal |
| Recovery | Account re-enablement, MFA enforcement, post-recovery monitoring |
| Post-Incident | Root cause analysis, lessons learned, security improvements |

### CIS Controls v8

| Control | Relevance |
|---|---|
| CIS 4 - Secure Configuration | MFA enforcement via Conditional Access |
| CIS 5 - Account Management | Session revocation, password reset, access review |
| CIS 6 - Access Control Management | Conditional Access policy implementation |
| CIS 8 - Audit Log Management | Entra ID and Exchange audit log review |
| CIS 17 - Incident Response | Full IR lifecycle documented |

### Microsoft Security Benchmark

| Category | Control Applied |
|---|---|
| Identity Management | MFA enforcement, Conditional Access |
| Privileged Access | Session revocation, least-privilege validation |
| Logging & Monitoring | Sign-in logs, mailbox audit, message trace |
| Incident Response | Containment, remediation, recovery documented |

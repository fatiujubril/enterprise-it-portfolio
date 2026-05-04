# Detection Rules Index

| # | Rule Name | MITRE Technique | Tactic | Severity | Data Source | FP Rate Before | FP Rate After | Status |
|---|---|---|---|---|---|---|---|---|
| 1 | Impossible Travel | T1078.004 | Initial Access | High | SigninLogs | - | - | 🔴 Pending |
| 2 | Brute Force Detection | T1110 | Credential Access | High | SigninLogs | - | - | 🔴 Pending |
| 3 | MFA Fatigue | T1621 | Credential Access | High | SigninLogs | - | - | 🔴 Pending |
| 4 | Suspicious App Consent | T1528 | Credential Access | Medium | AuditLogs | - | - | 🔴 Pending |
| 5 | New Admin Role Assignment | T1098 | Privilege Escalation | High | AuditLogs | - | - | 🔴 Pending |
| 6 | Bulk User Deletion | T1531 | Impact | High | AuditLogs | - | - | 🔴 Pending |
| 7 | Legacy Auth Sign-in | T1078 | Initial Access | Medium | SigninLogs | - | - | 🔴 Pending |
| 8 | After Hours Sign-in | T1078 | Initial Access | Low | SigninLogs | - | - | 🔴 Pending |
| 9 | Service Principal Anomaly | T1528 | Persistence | Medium | AADServicePrincipalSignInLogs | - | - | 🔴 Pending |
| 10 | Account Manipulation | T1098.001 | Privilege Escalation | High | AuditLogs | - | - | 🔴 Pending |

---

## Status Legend
- 🔴 Pending — not yet built
- 🟡 In Progress — query written, not yet tuned
- 🟢 Complete — query written, tested, tuned, documented

# MITRE ATT&CK Mapping

## Techniques Covered

| Phase | ID | Technique | Implementation | Detection |
|---|---|---|---|---|
| Defense Evasion | T1556 | Modify Authentication Process | CA exception downgrades auth requirement | Sign-in logs - CA Not Applied |
| Defense Evasion | T1556.006 | Multi-Factor Authentication | MFA bypassed via policy exclusion | Auth requirement field - single-factor |
| Initial Access | T1078 | Valid Accounts | High-value account accessible with reduced auth | Sign-in anomaly detection |
| Credential Access | T1621 | MFA Request Generation | MFA challenge never issued due to bypass | CA evaluation result |

## Tactic Coverage

| Tactic | Coverage |
|---|---|
| Initial Access | T1078 |
| Defense Evasion | T1556, T1556.006 |
| Credential Access | T1621 |
| Persistence | Exception with no expiration creates persistent exposure |

## Detection Coverage Assessment

| Technique | Detected in Sim? | Method | Automated? |
|---|---|---|---|
| T1556 | Yes - manual review | Sign-in log analysis | No |
| T1556.006 | Yes - manual review | Auth requirement field | No |
| T1078 | Yes - manual review | CA status field | No |

## Detection Gap

No automated detection fired during this incident. All detection was manual.

Implementing the KQL queries in `03-Detection-Analysis/kql-queries.md` as
Sentinel analytics rules would convert this from reactive to proactive detection.

## ATT&CK Navigator Note

This simulation maps primarily to the Identity sub-techniques of T1556,
which are often overlooked in traditional SOC detection coverage focused
on endpoint and network telemetry.

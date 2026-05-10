# Evidence Index

All screenshots stored in Evidence/screenshots/ organized by phase.
Files are numbered sequentially for easy cross-reference.

## Baseline Evidence

| # | File | Description |
|---|---|---|
| 01 | [01-P2-BL-ENTRA-Tenant-Overview.png](screenshots/baseline/01-P2-BL-ENTRA-Tenant-Overview.png) | Entra ID tenant overview and licensing at baseline |
| 02 | [02-P2-BL-CA-Policy-List.png](screenshots/baseline/02-P2-BL-CA-Policy-List.png) | Active CA policies - MFA enforcement confirmed |
| 03 | [03-P2-BL-Auth-Methods-Configuration.png](screenshots/baseline/03-P2-BL-Auth-Methods-Configuration.png) | Authentication methods - Authenticator enabled |
| 04 | [04-P2-BL-SignIn-Logs-Clean.png](screenshots/baseline/04-P2-BL-SignIn-Logs-Clean.png) | Baseline sign-in logs - MFA enforced, no anomalies |

## Attack Simulation Evidence

| # | File | Description |
|---|---|---|
| 05 | [05-P2-AT-CA-Users-Group-Membership.png](screenshots/attack/05-P2-AT-CA-Users-Group-Membership.png) | CA policy user scope - group membership |
| 06 | [06-P2-AT-CA-Policy-Targets-Group.png](screenshots/attack/06-P2-AT-CA-Policy-Targets-Group.png) | CA policy targeting configuration |
| 07 | [07-P2-AT-CA-Grant-Require-MFA.png](screenshots/attack/07-P2-AT-CA-Grant-Require-MFA.png) | CA grant control requiring MFA |
| 08 | [08-P2-AT-CA-User-Excluded.png](screenshots/attack/08-P2-AT-CA-User-Excluded.png) | CEO account explicitly excluded from CA policy |

## Detection Evidence

| # | File | Description |
|---|---|---|
| 09 | [09-P2-DE-SignIn-Logs-MFA-Bypass.png](screenshots/detection/09-P2-DE-SignIn-Logs-MFA-Bypass.png) | Sign-in logs - CA Not Applied, single-factor auth on CEO |
| 10 | [10-P2-DE-CA-Policy-Config.png](screenshots/detection/10-P2-DE-CA-Policy-Config.png) | CA policy configuration review |
| 11 | [11-P2-DE-CA-Policy-Explicit-Exclusion.png](screenshots/detection/11-P2-DE-CA-Policy-Explicit-Exclusion.png) | CA policy exclusion confirmed as root cause |

## Incident Response Evidence

| # | File | Description |
|---|---|---|
| 12 | [12-P2-IR-CA-Exclusion-Removed.png](screenshots/incident-response/12-P2-IR-CA-Exclusion-Removed.png) | CA exclusion removed - CEO no longer excluded |
| 13 | [13-P2-IR-CA-MFA-Enforced.png](screenshots/incident-response/13-P2-IR-CA-MFA-Enforced.png) | MFA enforcement confirmed post-containment |

## Recovery Evidence

| # | File | Description |
|---|---|---|
| 14 | [14-P2-RC-CEO-MFA-Authenticator-Enabled.png](screenshots/recovery/14-P2-RC-CEO-MFA-Authenticator-Enabled.png) | MFA re-enrolled on CEO new mobile device |
| 15 | [15-P2-RC-Post-Remediation-MFA-Enforced.png](screenshots/recovery/15-P2-RC-Post-Remediation-MFA-Enforced.png) | Post-recovery sign-in - CA Success, MFA enforced |

## Evidence Handling Notes

- All screenshots captured in a controlled lab tenant
- No real user data or client information included
- Files numbered sequentially to support phase-based narrative
- Phase prefix codes: BL=Baseline, AT=Attack, DE=Detection, IR=Incident Response, RC=Recovery

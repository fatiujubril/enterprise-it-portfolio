# Compliance Notes

## NIST SP 800-61 (Incident Response)

| Phase | Coverage in This Project |
|---|---|
| Preparation | Baseline CA config documented, logging architecture defined |
| Detection & Analysis | Sign-in telemetry review, CA evaluation correlation |
| Containment | CA exception removed, enforcement immediately restored |
| Eradication | MFA re-enrollment resolves root usability issue |
| Recovery | Post-recovery validation, telemetry confirms enforcement |
| Post-Incident | Root cause analysis, governance recommendations documented |

## CIS Controls v8

| Control | Application |
|---|---|
| CIS 4 - Secure Configuration | CA policy design and exception governance |
| CIS 5 - Account Management | High-value identity protection, exception review |
| CIS 6 - Access Control Management | MFA enforcement via Conditional Access |
| CIS 8 - Audit Log Management | Entra ID sign-in and audit log review |
| CIS 17 - Incident Response | Full IR lifecycle documented and executed |

## Microsoft Zero Trust Framework

| Principle | Application |
|---|---|
| Verify Explicitly | CA policy validates identity + MFA on every sign-in |
| Use Least Privilege | Exception governance limits who can bypass controls |
| Assume Breach | Sign-in telemetry treated as primary detection signal |

## Microsoft Identity Security Benchmark

| Category | Control |
|---|---|
| Identity Management | MFA enforcement, CA policy design |
| Privileged Access | High-value identity exception governance |
| Logging and Monitoring | Sign-in logs, CA evaluation results |
| Incident Response | Detection, containment, recovery lifecycle |

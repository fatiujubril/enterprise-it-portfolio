# Logging Architecture

## Logging Scope

This project focused on identity-layer visibility only. The following
Microsoft Entra ID logging sources were enabled and used throughout:

| Log Source | Purpose |
|---|---|
| Sign-in Logs (Interactive) | Detect authentication anomalies and MFA bypass |
| Sign-in Logs (Non-Interactive) | Background authentication context |
| Audit Logs | Track CA policy changes and exclusions |
| CA Evaluation Results | Confirm whether policies applied or were bypassed |

## Key Detection Fields

| Field | Significance |
|---|---|
| ConditionalAccessStatus | Success vs Not Applied - primary bypass indicator |
| AuthenticationRequirement | MFA vs single-factor - confirms bypass impact |
| UserPrincipalName | Identifies affected account |
| IPAddress / Location | Contextual risk signal |
| ResultType | 0 = success, non-zero = failure |

## Detection Signal

The absence of CA enforcement (Not Applied) rather than a policy failure
was the core detection indicator in this incident. This is a critical
distinction - the policy existed and appeared active, but was silently
bypassed due to the exclusion.

## Logging Gaps Identified

| Gap | Risk |
|---|---|
| No alert for high-value account CA exclusion | High - exclusion went undetected |
| No automatic expiration on CA exceptions | High - indefinite exposure window |
| No alert when CA evaluates Not Applied for privileged users | High - bypass was silent |

## Logging Design Assumptions

- Cloud-native identity-only environment
- No SIEM ingestion - manual portal analysis only
- This mirrors many small-to-mid Microsoft 365 tenants

This constraint makes the detection methodology more broadly applicable -
these signals are available in every Microsoft 365 tenant without Sentinel.

# Misconfiguration Details

## What Was Misconfigured

The CA-Baseline-Require-MFA-All-Users Conditional Access policy was modified
to explicitly exclude the CEO user account from its scope.

This single change caused Conditional Access to evaluate as Not Applied for
all subsequent CEO sign-ins, silently downgrading authentication from MFA
to single-factor.

## Misconfiguration Anatomy

| Setting | Secure State | Misconfigured State |
|---|---|---|
| **Policy Users** | All users (group) | All users + CEO excluded |
| **CA Evaluation for CEO** | Success - MFA required | Not Applied |
| **Authentication Requirement** | Multifactor authentication | Single-factor authentication |
| **Exclusion Expiration** | N/A | None configured |
| **Exclusion Review** | N/A | None configured |

## Why This Is High Risk

1. **Silent bypass** - no error, no alert, policy appears active
2. **High-value target** - CEO account has broad access to sensitive data
3. **No time limit** - exception persists indefinitely without governance
4. **Difficult to detect** - requires active telemetry review, not config review

## The Governance Failure

The misconfiguration itself is not unusual. The failure is in the governance:
- No approval workflow requiring security sign-off
- No time-bound expiration on the exclusion
- No automated alert when CA evaluates Not Applied
- No periodic review process for active CA exceptions

## Evidence

| # | Screenshot | Description |
|---|---|---|
| 05 | [05-P2-AT-CA-Users-Group-Membership.png](../Evidence/screenshots/attack/05-P2-AT-CA-Users-Group-Membership.png) | CA policy user scope - group membership |
| 06 | [06-P2-AT-CA-Policy-Targets-Group.png](../Evidence/screenshots/attack/06-P2-AT-CA-Policy-Targets-Group.png) | CA policy targeting configuration |
| 07 | [07-P2-AT-CA-Grant-Require-MFA.png](../Evidence/screenshots/attack/07-P2-AT-CA-Grant-Require-MFA.png) | CA grant control - MFA required |
| 08 | [08-P2-AT-CA-User-Excluded.png](../Evidence/screenshots/attack/08-P2-AT-CA-User-Excluded.png) | CEO account explicitly excluded from CA policy |

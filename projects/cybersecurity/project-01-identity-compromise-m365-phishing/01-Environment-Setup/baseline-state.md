# Baseline State

## Pre-Incident Validation

Before simulating the attack, the environment was validated to confirm a clean baseline.
This ensures all subsequent activity can be confidently attributed to the simulated compromise.

## Confirmed Baseline Conditions

| Check | Result |
|---|---|
| User group membership | Standard domain user only - no admin groups |
| Local admin privileges | None |
| Domain admin privileges | None |
| Mailbox rules | None configured |
| Email forwarding | Disabled |
| Recent sign-in activity | None - no anomalies |
| MFA status | Registered, not enforced |

## Evidence

- [01-ad-user-group-membership.png](../Evidence/screenshots/01-ad-user-group-membership.png)
- [02-endpoint-identity-network.png](../Evidence/screenshots/02-endpoint-identity-network.png)
- [03-endpoint-privilege-groups.png](../Evidence/screenshots/03-endpoint-privilege-groups.png)
- [04-mailbox-rules-baseline.png](../Evidence/screenshots/04-mailbox-rules-baseline.png)
- [05-mailbox-forwarding-baseline.png](../Evidence/screenshots/05-mailbox-forwarding-baseline.png)

## Why This Matters

Establishing a clean baseline is a critical step in any incident simulation or investigation.
It provides the reference point against which all attacker activity is compared, and ensures
findings are attributable to the simulated compromise rather than pre-existing conditions.

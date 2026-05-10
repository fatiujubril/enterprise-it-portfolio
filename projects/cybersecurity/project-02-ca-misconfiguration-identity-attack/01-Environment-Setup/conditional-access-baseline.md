# Conditional Access Baseline

## Baseline Policy Design

At baseline, Conditional Access was configured to enforce MFA consistently
across the tenant using a single, clearly scoped policy.

**Design principles:**
- MFA enforcement for all interactive user sign-ins
- Group-based targeting for scalability and governance
- No user, role, or device-based exclusions
- Policy enabled and actively enforcing (not report-only)

## Baseline Policy Configuration

| Setting | Value |
|---|---|
| **Policy Name** | CA-Baseline-Require-MFA-All-Users |
| **State** | Enabled |
| **Users** | All users (group-scoped) |
| **Cloud Apps** | All cloud apps |
| **Conditions** | Any location, any device |
| **Grant Control** | Require MFA |
| **Exclusions** | None at baseline |

## Enforcement State at Baseline

Before any misconfiguration:
- CA evaluated successfully for all users
- MFA challenges consistently applied during sign-in
- No accounts bypassed authentication requirements
- No emergency or exception-based access paths present

## Baseline Evidence

| # | Screenshot | Description |
|---|---|---|
| 02 | [02-P2-BL-CA-Policy-List.png](../Evidence/screenshots/baseline/02-P2-BL-CA-Policy-List.png) | Active CA policies at baseline - MFA enforcement confirmed |

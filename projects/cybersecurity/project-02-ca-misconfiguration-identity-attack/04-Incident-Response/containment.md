# Containment - CA Exception Removal

## Containment Objective

Immediately eliminate the active MFA bypass risk by removing the CA exception,
without disrupting the broader tenant or other user accounts.

## Containment Actions

### Action 1 - Remove CA Policy Exclusion

The CEO account was removed from the exclusion list of the
`CA-Baseline-Require-MFA-All-Users` policy.

- Policy remained enabled for all other users throughout
- No report-only state or policy rollback used
- Removal triggered immediate CA re-evaluation on next sign-in

### Action 2 - Verify Enforcement Restored

Post-removal sign-in attempts confirmed:
- CA status: Success (previously Not Applied)
- Authentication requirement: Multifactor authentication

## Containment Evidence

| # | Screenshot | Description |
|---|---|---|
| IR-01 | 12-P2-IR-CA-Exclusion-Removed.png | CA exclusion removed - CEO no longer in exclusion list |

## Containment Outcome

- CA enforcement restored for affected account
- MFA requirements applied to all interactive sign-ins
- No additional excluded accounts identified
- Active MFA bypass risk neutralized

## Sequencing Note

Containment was executed before MFA re-enrollment to close the bypass window
immediately. A brief period of restricted access for the CEO was accepted as
a necessary trade-off to eliminate the security exposure.

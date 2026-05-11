# Containment - CA Exception Removal

## Containment Objective

Immediately eliminate the active MFA bypass risk by removing the CA exception,
without disrupting the broader tenant or other user accounts.

---

## Containment Actions

### Action 1 - Remove CA Policy Exclusion

The CEO account was removed from the exclusion list of the
CA-Baseline-Require-MFA-All-Users policy.

- Policy remained enabled for all other users throughout
- No report-only state or policy rollback used
- Removal triggered immediate CA re-evaluation on next sign-in

### Action 2 - Verify Exclusion Tab Is Clean

Post-removal review of the CA policy Exclude tab confirmed:
- No users or groups listed under exclusions
- No directory roles excluded
- No guest or external user exemptions

This confirms the exclusion was fully removed, not just disabled.

---

## Containment Evidence

| # | Screenshot | Description |
|---|---|---|
| 12 | [12-P2-IR-CA-Exclusion-Removed.png](../Evidence/screenshots/incident-response/12-P2-IR-CA-Exclusion-Removed.png) | CA policy Exclude tab - all checkboxes empty, exclusion confirmed removed |

---

## Containment Outcome

- CA enforcement restored for affected account
- MFA requirements applied to all interactive sign-ins
- No additional excluded accounts identified
- Active MFA bypass risk neutralized

## Sequencing Note

Containment was executed before MFA re-enrollment to close the bypass window
immediately. A brief period of restricted CEO access was accepted as a
necessary trade-off to eliminate the active security exposure.

> Full before/after telemetry validation is documented in remediation.md
> with screenshot 13 showing the sign-in log comparison.

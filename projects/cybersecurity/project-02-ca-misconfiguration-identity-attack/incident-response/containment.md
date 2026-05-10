# Incident Response – Containment

This section documents the containment actions taken after the detection of a Conditional Access misconfiguration that allowed single-factor authentication for a high-value user account.

The objective of containment was to **immediately eliminate the active risk**, prevent further MFA bypass, and restore intended Conditional Access enforcement.

---

## Containment Objective

- Stop ongoing single-factor authentication access
- Remove the Conditional Access exception responsible for the bypass
- Ensure policy enforcement is re-applied consistently
- Prevent further exposure while remediation activities are prepared

---

## Containment Actions Taken

Upon confirmation that the Chief Executive Officer (CEO) account was authenticating without MFA, the following containment actions were executed:

1. **Conditional Access Exception Removal**
   - The CEO account was removed from the exclusion list of the `CA-Baseline-Require-MFA-All-Users` policy
   - The policy remained enabled and actively enforcing MFA requirements

2. **Immediate Policy Re-Evaluation**
   - Removal of the exclusion triggered Conditional Access re-evaluation on subsequent sign-ins
   - No policy rollback or report-only state was used

3. **Access Restriction Restored**
   - Post-containment sign-ins required MFA completion
   - Single-factor authentication access was no longer permitted

These actions effectively closed the active identity exposure window.

---

## Containment Evidence

### Evidence 5.1 – Conditional Access Exception Removed

This evidence confirms that the explicit Conditional Access exclusion responsible for the MFA bypass was removed from the policy configuration.

![Evidence 5.1 – Conditional Access Exception Removed](../evidence/screenshots/incident-response/P2-IR-CA-Exclusion-Removed-01.png)

---

## Containment Outcome

- Conditional Access enforcement was restored for the affected account
- MFA requirements were consistently applied across all interactive sign-ins
- No additional users were found to be excluded from the policy
- The immediate risk associated with the misconfiguration was neutralized

---

## Transition to Remediation

With the active risk contained, remediation efforts focused on **restoring secure normal operations**, including re-establishing a functional MFA method for the affected user and validating long-term enforcement.

Subsequent remediation and recovery actions are documented in the following sections.

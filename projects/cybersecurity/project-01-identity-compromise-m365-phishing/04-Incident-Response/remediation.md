# Remediation

## Remediation Objective

Remove all attacker-established persistence mechanisms and restore the account
to a secure state with stronger controls than existed before the compromise.

## Remediation Actions

### Action 1 - Password Reset

**What:** Forced password reset with change required on next login.
**Why:** Invalidates the stolen credentials used for initial access.

Evidence: [12-password-reset.png](../Evidence/screenshots/12-password-reset.png)

---

### Action 2 - Malicious Inbox Rule Removal

**What:** Removed the inbox rule created by the attacker.
**Why:** Eliminates persistence mechanism and restores full visibility of inbox.

Steps taken:
1. Navigate to Exchange Admin Center > Mailboxes > [Affected User]
2. Review all inbox rules
3. Identify and remove the malicious rule
4. Confirm removal

Evidence: [13-mailbox-rule-removed.png](../Evidence/screenshots/13-mailbox-rule-removed.png)

---

### Action 3 - MFA Enforcement via Conditional Access

**What:** Created a Conditional Access policy requiring MFA for all cloud app access.
**Why:** Eliminates the primary control gap that allowed single-factor authentication.

Policy configuration:
- Users: All users (or targeted group)
- Cloud apps: All cloud apps
- Conditions: Any location
- Grant: Require multi-factor authentication

Evidence: [14-mfa-enabled.png](../Evidence/screenshots/14-mfa-enabled.png)

---

## Remediation Verification

Post-remediation checks confirmed:
- Attacker credentials are no longer valid
- No malicious inbox rules remain
- MFA is now enforced and cannot be bypassed
- User account restored to normal operation

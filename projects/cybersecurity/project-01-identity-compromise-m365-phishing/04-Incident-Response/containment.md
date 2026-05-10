# Containment

## Containment Objective

Terminate attacker access and invalidate all active sessions without destroying
investigative evidence or disrupting user recovery.

## Containment Actions

### Action 1 - Session Revocation

**What:** Revoked all active sessions for the compromised account via Entra ID.
**Why:** Invalidates attacker tokens immediately, terminating active OWA access.
**Sequencing note:** Performed before password reset to prevent token reuse.

Steps taken:
1. Navigate to Entra ID > Users > [Affected User]
2. Select "Revoke sessions"
3. Confirm revocation
4. Verify no active sessions remain

Evidence: [11-session-revocation.png](../Evidence/screenshots/11-session-revocation.png)

---

### Action 2 - Account Disable (Temporary)

**What:** Disabled the user account during the active investigation window.
**Why:** Prevents re-authentication while investigation and remediation are completed.

---

## Containment Sequencing Rationale

Session revocation was performed before password reset deliberately.
If the password is reset first without revoking sessions, the attacker's existing
tokens remain valid and can continue to be used until they expire naturally.
Revoking sessions first closes this window completely.

## Evidence Preservation Note

All log evidence was captured and documented before containment actions were taken.
Containment does not overwrite or destroy Entra ID sign-in logs or Exchange audit logs.

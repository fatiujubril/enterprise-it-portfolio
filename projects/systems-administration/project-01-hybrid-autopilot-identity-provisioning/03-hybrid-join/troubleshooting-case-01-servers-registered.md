# Phase 3 — Troubleshooting Case 01: Servers and Domain Controller Unintentionally Registered as Entra Devices

### Symptom

After configuring hybrid Azure AD join, the Entra ID Devices list showed not only the intended client (`WIN11-USER01`) but also **DC01** and **SYNC01**, both listed as Microsoft Entra hybrid joined with a "Pending" registration state. Infrastructure servers had acquired cloud device identities that they should not have.

📸 [Symptom — DC01 and SYNC01 registered alongside the client](./screenshots/breakfix-01-servers-registered-unintended.png)

### Root Cause

Hybrid Azure AD join is configured at the **domain level** — the Service Connection Point applies to the entire forest, and the `Automatic-Device-Join` scheduled task runs on **every** domain-joined Windows machine. Domain controllers and member servers are domain-joined Windows machines, so they picked up the SCP and attempted to register just as a client would.

This is expected default behaviour, but undesirable: device registration is a **client-endpoint feature**. It exists so user-facing devices can be managed by Intune and evaluated by Conditional Access. Servers are managed by other means and have no need for a cloud device identity. Registering domain controllers in particular is explicitly discouraged by Microsoft — a DC is Tier-0 infrastructure, and giving it a cloud device identity needlessly expands the attack surface on the most sensitive system in the environment.

### Resolution

The fix has two parts: **prevent** future registration via Group Policy, and **remove** the existing registrations. Both are required — Group Policy prevents recurrence but does not un-register a device that has already joined.

**1. Prevent — Group Policy to disable device registration on servers and DCs.**

A GPO (`Disable Device Registration - Servers and DCs`) was created with the setting **Computer Configuration → Administrative Templates → Windows Components → Device Registration → "Register domain-joined computers as devices"** set to **Disabled**.

📸 [GPO setting disabled](./screenshots/breakfix-02-gpo-disable-device-registration.png)

**2. Scope the GPO correctly — a diagnostic sub-issue.**

Initial verification with `gpresult /r /scope:computer` showed the GPO listed under both "Applied Group Policy Objects" *and* "filtered out... Disabled (Link)." Investigation revealed the GPO had two links: one at the domain root (link disabled) and one at the `Lab-Servers` OU (link enabled). The disabled domain-root link was creating the confusing "Disabled (Link)" entry.

The domain-root link was removed (it would also have incorrectly targeted client machines), leaving clean links scoped only to the **Domain Controllers** OU and the **Lab-Servers** OU. SYNC01 was confirmed to reside in `OU=Lab-Servers`, so the GPO reaches it.

After cleanup, `gpresult` showed the GPO cleanly applied with no filtered-out entry:

📸 [gpresult — GPO cleanly applied](./screenshots/breakfix-03-gpresult-link-enabled.png)

**3. Remove — clear existing registrations with `dsregcmd /leave`.**

Because the GPO only prevents future registration, `dsregcmd /leave` was run on both servers to remove their existing Entra device registration. Both then showed `AzureAdJoined : NO` while correctly retaining `DomainJoined : YES` — back to being plain domain members.

📸 [SYNC01 after leave — AzureAdJoined NO](./screenshots/breakfix-04-dsregcmd-sync01-after.png)

📸 [DC01 after leave — AzureAdJoined NO](./screenshots/breakfix-05-dsregcmd-dc01-after.png)

**4. Clean up orphaned device objects.**

The now-orphaned DC01 and SYNC01 device objects were deleted from the Entra Devices list, leaving only the intended client:

📸 [Only WIN11-USER01 remains in Entra](./screenshots/breakfix-06-servers-removed-from-entra.png)

### Verification

- Both servers report `AzureAdJoined : NO`, `DomainJoined : YES`.
- `gpresult /r /scope:computer` on both servers shows the GPO applied with no filtered-out entry.
- The Entra Devices list contains only `WIN11-USER01`.
- The client (`WIN11-USER01`) is in the `Lab-Computers` OU, which the GPO does not target, so it continues to hybrid-join normally.

### Lesson / Takeaway

Two principles came out of this:

1. **Device registration is a client feature and must be scoped.** Hybrid join is domain-wide by default; without scoping, every server and domain controller registers. The correct design excludes Tier-0 infrastructure and member servers from device registration — DCs especially should never be device-registered.
2. **A GPO appearing in "Applied Group Policy Objects" is not sufficient proof it is enforcing** — the link must also be enabled and correctly scoped. Reading `gpresult` carefully enough to spot a disabled link is what distinguishes a real verification from a superficial one.

The mental model: **Group Policy prevents, `dsregcmd /leave` removes** — remediation of an already-registered device requires both.

> **Follow-up:** this remediation later proved incomplete — the records reappeared days afterward. See [Case 02](./troubleshooting-case-02-usercertificate-resync.md) for the recurrence, the differential diagnosis, and the completed fix.

---

[← Back to Phase 3 README](./README.md) | [Project overview](../README.md)

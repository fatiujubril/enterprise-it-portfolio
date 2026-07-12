# Troubleshooting Notes — Phase 3

## Case 01 — Servers and Domain Controller Unintentionally Registered as Entra Devices

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

> **Follow-up:** this remediation later proved incomplete — the records reappeared days afterward. See **Case 02** below for the recurrence, the differential diagnosis, and the completed fix.

---

## Case 02 — DC01 and SYNC01 Device Records Reappear (Follow-Up to Case 01)

### Symptom

Days after the Case 01 remediation, the SYNC01 and DC01 device records **reappeared** in Entra ID → Devices, again showing as Microsoft Entra hybrid joined. Discovered incidentally while investigating an unrelated Autopilot item during Phase 4.

📸 [Symptom — both servers back in the Entra device list](./screenshots/breakfix-07-entra-devices-reappeared.png)

First read: the Case 01 GPO failed and the servers re-registered. That read turned out to be wrong — and the diagnostic that disproves it is the most important part of this case.

### Root Cause

Two different mechanisms can put a hybrid device record into Entra, with identical symptoms but entirely different fixes:

- **Mechanism A — the device actually re-registered.** The `Automatic-Device-Join` task ran and completed, meaning the GPO is not applying — the Case 01 failure recurring.
- **Mechanism B — Entra Connect re-created the records from stale AD data.** When a device registers, a certificate is written to the **`userCertificate` attribute of its AD computer object**. The sync engine exports computer objects carrying that attribute as device records. If the attribute survives a cleanup, the next sync cycles quietly re-seed the directory with stubs — nothing re-registered; the directory is echoing leftover source data.

The differentiator is local registration state:

```
dsregcmd /status
```

Both servers reported `AzureAdJoined : NO` — no local registration state exists, so nothing re-registered. The GPO is acquitted; this is **Mechanism B**.

📸 [dsregcmd — no local registration state on either server](./screenshots/breakfix-08-dsregcmd-azureadjoined-no.png)

Confirmed directly in ADUC (Attribute Editor on the computer objects): `userCertificate` was still populated with the DER-encoded certificate blob from the original registration.

📸 [Stale userCertificate still present on the computer object](./screenshots/breakfix-09-usercertificate-populated.png)

Root cause: **the Case 01 remediation was incomplete.** Hybrid device registration state lives in **three places** — the local machine (dsregcmd state), the Entra directory (the device object), and the AD computer object's `userCertificate` attribute. Case 01 cleaned the first two (`dsregcmd /leave` + Entra record deletion) but not the third, and the sync engine re-exported the stubs on a later cycle.

### Resolution

**1. Clear the stale attribute at the source.** In ADUC (Attribute Editor), the `userCertificate` attribute was cleared on both the SYNC01 and DC01 computer objects.

📸 [userCertificate cleared](./screenshots/breakfix-10-usercertificate-cleared.png)

**2. Force a sync cycle.** On SYNC01 (the cmdlet ships with Entra Connect and exists only on the Connect server):

```powershell
Start-ADSyncSyncCycle -PolicyType Delta
```

📸 [Delta sync — Result: Success](./screenshots/breakfix-11-delta-sync-success.png)

**3. No manual deletion required this time.** With the attribute cleared, both computer objects fell out of the sync engine's device-export scope and the sync engine removed the two Entra device records automatically on the cycle.

### Verification

- Entra ID → Devices: SYNC01 and DC01 records gone after the delta cycle.

  📸 [Clean device list post-sync](./screenshots/breakfix-12-entra-devices-clean.png)

- Re-checked after subsequent scheduled sync cycles (every 30 minutes): the records **stayed** gone — the persistence check Case 01 never performed.
- GPO health re-confirmed by the diagnosis itself: `AzureAdJoined : NO` on both servers means the registration block has held throughout.

### Lesson / Takeaway

**Hybrid device registration state lives in three places — the local machine, the Entra directory, and the AD computer object's `userCertificate` attribute. Remediation is only complete when all three are cleaned, and only verified when the symptom stays gone across sync cycles.**

Three principles came out of this:

1. **A recurring symptom does not mean the original control failed.** The differential diagnosis (`dsregcmd /status`) separated "control failed" (needs a GPO fix) from "cleanup incomplete" (needs an attribute clear) — two entirely different fixes for one identical symptom. Re-doing the Case 01 fix without diagnosing first would have "worked" for a few days and recurred again.
2. **Sync engines faithfully reproduce whatever the source says.** Cleaning a downstream copy (the Entra record) without cleaning the authoritative source (the AD attribute) guarantees reappearance. Stale data does not stay quietly stale — it propagates.
3. **"Gone" and "stayed gone" are different claims.** Verification of a fix should outlive the immediate moment; only the second claim closes an incident.

The completed mental model, extending Case 01: **Group Policy prevents, `dsregcmd /leave` removes locally, deleting the Entra object removes the copy — and clearing `userCertificate` removes the source the sync engine would rebuild it from.**

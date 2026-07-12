# Phase 3 — Troubleshooting Case 02: DC01 and SYNC01 Device Records Reappear

*Follow-up to [Case 01 — Servers and Domain Controller Unintentionally Registered as Entra Devices](./troubleshooting-case-01-servers-registered.md).*

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

---

[← Back to Phase 3 README](./README.md) | [Case 01](./troubleshooting-case-01-servers-registered.md) | [Project overview](../README.md)

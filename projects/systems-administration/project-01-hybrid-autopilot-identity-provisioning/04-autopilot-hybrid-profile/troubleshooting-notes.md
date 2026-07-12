# Phase 4 — Troubleshooting Notes

## breakfix-01 — Intune Connector enrollment sign-in fails with a generic error

### Symptom

During Intune Connector for Active Directory enrollment on SYNC01, the wizard's **Sign In** step failed immediately with the generic Entra sign-in error:

> **Something went wrong**
> An unanticipated error occurred. Your IT department may be able to help.

with only an Activity Id / Session Id / Timestamp for diagnostics — no actionable message.

📸 [Sign-in failure in the connector wizard](./screenshots/breakfix-01-connector-signin-error.png)

### Diagnosis

The account used was `cloudadmin@fjlab.ca`, which holds the **Intune Administrator** role — the prerequisite most documentation emphasizes. The updated (MSA-based) connector's enrollment account, however, has **two** requirements:

1. Intune Administrator (or Global Administrator) role ✔ — held
2. **An assigned Intune license** ✘ — missing; only the pilot end user had been licensed at this point

The wizard performs its account checks behind the embedded sign-in flow and surfaces any failure as this same generic dialog, which is what makes the root cause non-obvious. (The candidate list worked through, in order of likelihood for this environment: missing Intune license on the enrolling account → actual exception in Event Viewer under `Applications and Services Logs → Microsoft → Windows → IntuneODJConnector` → clock skew on the server (`w32tm /query /status`) → missing WebView2 runtime → cloud-only vs synced enrolling account, a legacy-connector-era report.)

### Resolution

1. Assigned an M365 E5 license to `cloudadmin@fjlab.ca` (usage location must be set on the account before license assignment succeeds).
2. Waited for license propagation (~5–10 minutes).
3. **Fully closed and relaunched** the enrollment wizard — it does not recover from the error state within a session.
4. Signed in again: enrollment succeeded, MSA created.

### Verification

- Wizard reported successful enrollment and MSA setup (`msaODJtLfqK$`).
- Connector status **Active** in Intune admin center → Devices → Enrollment → Windows → Intune Connector for Active Directory.
- MSA delegation verified directly in the Lab-Computers OU ACL (see [README, Step 4](./README.md)).

### Takeaway

**The connector enrollment prerequisite is role AND license, and the wizard fails with an unhelpful generic error when either is missing.** The license requirement is easy to miss because admin accounts are often deliberately unlicensed (a reasonable cost practice), and because the error gives no hint. The enrollment account is only needed at install time, so the license can be assigned temporarily and reclaimed afterward. Secondary lesson: license propagation is not instant — a correct fix can still "fail" if retested immediately, so build in a propagation wait before concluding a fix didn't work.

---

## breakfix-02 — Autopilot profile assignment stuck in "Assigning" for over 20 hours

### Symptom

After creating the deployment profile and assigning it to `AP-Devices-All`, the device's profile status moved **Not assigned → Pending → Assigning 'APHybridUserDriven'** — and then stopped. The device detail pane showed a Date assigned timestamp more than 20 hours old with the status still not reaching Assigned. Normal assignment completes in minutes to about an hour; a day is a stuck state, not latency.

📸 [Stuck state — Assigning, with a Date assigned timestamp from the previous day](./screenshots/breakfix-02-profile-stuck-assigning.png)

### Diagnosis

Profile assignment is an **entirely cloud-side transaction** between Intune and the Windows Autopilot deployment service, executed against the device's registration record. The physical device is not a participant — it can be powered off or sealed in a box; only at OOBE does the device download whatever profile the service has already attached. So the diagnosis targets the record and the services, never the device.

The candidate list worked through, in order:

1. **Dynamic group membership** — ruled out: assignment tables on profiles targeting `AP-Devices-All` showed `1 devices`, and the group's member list contained the device's Autopilot object.
2. **Forced service sync** — the Sync button on the Autopilot devices blade (rate-limited to ~15-minute intervals) was used multiple times across the stuck window with no effect.
3. **Profile-side cross-check** — the profile showed the device attached; the status column simply never progressed.
4. **Service health** — M365 admin center → Health → Service health showed Intune fully healthy; no Autopilot advisory to explain the stall.
5. **Hash validity (false alarm worth recording)** — inspection of the import CSV raised a scare: the hardware hash ended in a long run of `AAAA...` characters. This is **normal, not corruption**: the 4K hardware hash is a fixed-size binary structure, unpopulated fields are zero-filled, and zero bytes encode as `A` in Base64. A VM has far fewer populated hardware fields than OEM hardware, hence more padding. The populated portion contained genuine identity data (CPU model, virtual disk, vTPM version, UEFI release, serial), and Intune validates hashes at import — a malformed hash is rejected, not silently accepted.

With the record's inputs verified and the services healthy, the conclusion was corrupted/stalled assignment state on the registration record itself — for which the documented remediation is to reset the record.

### Resolution

1. **Deleted the device** from Windows Autopilot devices — then **waited until it fully disappeared** from the list before proceeding. Deletion is slow and processes in the background; re-importing while the old record still exists simply matches the existing hash and lands on the same stuck record, resetting nothing. (This also removes the associated placeholder Entra device object.)
2. **Re-imported the same hash CSV.** Nothing about the hardware had changed, so no re-capture was needed.
3. The re-import created a fresh registration with a **new ZTDId**; `AP-Devices-All` re-evaluated the new object in automatically (the dynamic rule matches any ZTDId, no group edits needed), and assignment re-ran against clean state.

### Verification

Profile status progressed **Not assigned → Pending → Assigned** on the normal timeline after the re-import — the same machinery that had hung for 20+ hours completed without intervention once the record was fresh.

📸 [Resolved — profile status Assigned after delete and re-import](./screenshots/15-autopilot-device-profile-assigned.png)

### Takeaway

**Autopilot profile assignment is a cloud-side transaction against the registration record, and that record's assignment state can wedge; when membership, sync, and service health all check out, delete-and-reimport is the reset — but only if the deletion fully completes before the re-import.**

Supporting lessons:

1. **The three assignment states are Not assigned → Pending → Assigning/Assigned, and the OOBE test must not run until Assigned.** A device provisioned while the profile is unattached gets vanilla consumer OOBE and looks like a total Autopilot failure when nothing is wrong.
2. **Know what a healthy hardware hash looks like before suspecting it.** Long Base64 `A` runs are zero-padding in a fixed-size structure — expected on VMs — not evidence of corruption. Chasing the hash would have been a dead end; recognizing the padding kept the diagnosis on track.
3. **Not every incident yields a satisfying root cause.** The honest conclusion is "service-side assignment state stalled; re-registration reset it." Documenting the *elimination process* (membership → sync → profile-side → service health → hash validity) is what demonstrates the diagnostic skill — and is what makes "we reset it" a defensible resolution rather than a shrug.

---

[← Back to Phase 4 README](./README.md) | [Project overview](../README.md)

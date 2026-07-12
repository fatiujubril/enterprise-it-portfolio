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

## Observation (not a failure) — Autopilot profile assignment latency

After creating the deployment profile and assigning it to `AP-Devices-All`, the device's profile status moved **Not assigned → Pending** and remained Pending for an extended period. This is documented service behavior, not a fault: profile assignment is an asynchronous background job between Intune and the Windows Autopilot deployment service, and can take minutes to hours (longest on a tenant's first-ever assignment).

Working diagnostic sequence before assuming a stuck state:

1. Confirm group membership actually evaluated (the assignment table on any profile targeting the group shows the device count — here it showed `1 devices`, ruling out a membership problem).
2. Use the **Sync** button on the Autopilot devices blade to force service-side processing (then blocked for ~15 minutes between syncs).
3. Cross-check from the profile side (profile → assigned devices) — the profile side often reflects the attachment before the device list column updates.
4. Only if still Pending after many hours: delete and re-import the Autopilot device to reset the assignment machinery (last resort — premature re-imports cause more problems than they solve).

Key rule: **do not run the OOBE test while the status is Pending.** The profile is not yet available to the device; the run produces a vanilla consumer OOBE and looks like a total Autopilot failure when nothing is actually wrong.

---

[← Back to Phase 4 README](./README.md) | [Project overview](../README.md)

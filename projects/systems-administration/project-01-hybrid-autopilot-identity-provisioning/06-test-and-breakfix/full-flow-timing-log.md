# Full-Flow Timing Log

Timing of the end-to-end Autopilot provisioning run, captured to characterize the flow and provide a baseline for future comparison (for example, after enabling Windows updates during OOBE, or after the network redesign).

Precise per-phase timestamps were not recorded during the run. The figures below are **estimates** apportioned against the observed total: the provisioning phases, excluding the break/fix, completed in **under 6 minutes** end to end — consistent with a small single-user tenant on a fast host.

> **Note:** this run included the domain-sign-in break/fix (see [breakfix-case-study.md](./breakfix-case-study.md)), which consumed the large majority of wall-clock time and is excluded from the totals below. The estimates characterize the *clean* provisioning path only.

## Run 1 — estimated phase breakdown (provisioning path, break/fix excluded)

| Phase | Est. duration | Notes |
|---|---|---|
| OOBE to work-or-school sign-in | ~1:00 | Includes the OOBE critical-update check and restart |
| Sign-in → ESP appears | ~0:20 | Credential validation, profile fetch |
| ESP: Device preparation | ~0:30 | |
| ESP: Device setup (ODJ round-trip) | ~1:30 | The hybrid-specific cost — connector creates `LAB-6KMEnXs2vpf`, device applies the offline domain join, reboot |
| First domain sign-in (post-ODJ) | ~0:30 | *This is where the break/fix occurred; the estimate reflects the successful post-fix sign-in* |
| ESP: Account setup (incl. re-auth) | ~1:30 | Second authentication prompt, profile creation |
| Account setup → desktop | ~0:20 | |
| **Estimated total (provisioning path)** | **~5:40** | Under 6 minutes, consistent with observation |

*Post-desktop:* the Finance app (Adobe Acrobat Reader) installed a few minutes **after** the desktop appeared, not during ESP — expected for a user-targeted Win32/Store app, which the Intune Management Extension processes once enrollment completes. Not counted in the provisioning total because the desktop was already usable.

## Observations

- **Device setup is the hybrid-specific cost.** The ODJ round-trip through the connector is time cloud-native Autopilot does not spend at all — a concrete illustration of what the hybrid model costs in exchange for the domain join.
- **Windows updates during OOBE were disabled** in the ESP for this first run to minimize variables. Enabling them — the production-appropriate setting — would add substantial time and at least one reboot, plausibly turning a ~6-minute run into 20–40+ minutes.
- **The break/fix consumed the vast majority of real wall-clock time.** The provisioning mechanics were quick once DNS resolved. This is the normal shape of a first hybrid run: the mechanics are fast, the environmental dependencies are where the time goes.
- **Measurement lesson:** timestamps should be captured live rather than reconstructed. A future run should record them at each phase transition — the estimates above are defensible against the observed total, but they are estimates, and saying so is more useful than presenting them as measurements.

## Run 2 — clean run on redesigned network (planned)

To be captured after the internal/private vSwitch redesign (see the project backlog), with precise timestamps, as the representative baseline — and ideally a second variant with OOBE Windows updates enabled, to quantify that cost.

| Phase | Start | End | Duration | Notes |
|---|---|---|---|---|
| | | | | |

---

[← Back to Phase 6 README](./README.md) | [Break/fix case study](./breakfix-case-study.md) | [Project overview](../README.md)

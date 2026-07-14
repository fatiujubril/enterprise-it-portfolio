# Full-Flow Timing Log

Rough timing of the end-to-end Autopilot provisioning run, captured to characterize the flow and provide a baseline for future comparison (e.g. after enabling Windows updates during OOBE, or after the network redesign). Times are approximate — the goal is proportion and phase identification, not stopwatch precision.

> **Note:** this run included the domain-sign-in break/fix (see [breakfix-case-study.md](./breakfix-case-study.md)), so wall-clock total is not representative of a clean run. Phase durations up to the failure, and after the fix, are still useful. A future clean run on the redesigned network should be logged alongside for comparison.

## Run 1 — with break/fix

| Phase | Start | End | Duration | Notes |
|---|---|---|---|---|
| OOBE to work-or-school sign-in | | | | includes an OOBE critical-update + restart |
| Sign-in → ESP appears | | | | |
| ESP: Device preparation | | | | |
| ESP: Device setup (ODJ round-trip) | | | | connector creates LAB-6KMEnXs2vpf, device applies join |
| Reboot after ODJ | | | | |
| First domain sign-in | | | | **FAILED here — see break/fix; DNS fix applied** |
| ESP: Account setup (incl. re-auth) | | | | Acrobat (Finance app) delivered post-desktop |
| Account setup → desktop | | | | |
| **Total (excluding break/fix time)** | | | | |

## Observations

- Device setup (the ODJ phase) is the hybrid-specific cost — cloud-native Autopilot has no equivalent connector round-trip.
- Windows updates during OOBE were **disabled** in the ESP for this first run to minimize variables; enabling them (production setting) would add substantial time and at least one reboot.
- The break/fix consumed the majority of wall-clock time; the actual provisioning phases were relatively quick once DNS resolved.
- The Finance app (Acrobat) installed **after** the desktop appeared, not during ESP — expected for a user-targeted Win32/Store-new app, which the Intune Management Extension processes once enrollment completes.

## Run 2 — clean run on redesigned network (planned)

To be captured after the internal/private vSwitch redesign, as the representative baseline.

| Phase | Start | End | Duration | Notes |
|---|---|---|---|---|
| | | | | |

---

[← Back to Phase 6 README](./README.md) | [Break/fix case study](./breakfix-case-study.md) | [Project overview](../README.md)

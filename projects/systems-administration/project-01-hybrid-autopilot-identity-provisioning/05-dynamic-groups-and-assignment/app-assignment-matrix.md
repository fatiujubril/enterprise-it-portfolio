# App Assignment Matrix — Department-Driven Application Delivery

## The concept

With department dynamic groups in place ([dynamic-group-rules.md](./dynamic-group-rules.md)), applications are assigned to those groups rather than to individual users or devices. A user's group membership — derived from their `department` attribute — determines which apps they receive, automatically, on whatever device they sign in to. This is identity-driven application delivery: the app follows the person, not the machine.

## Current assignment

| App | Type | Assignment | Target group | Intent |
|---|---|---|---|---|
| Adobe Acrobat Reader DC | Microsoft Store app (new) / Win32 | **Required** | `DYN-Users-Finance` | Baseline Finance productivity app; also serves as the end-to-end proof that user-targeted assignment reaches a provisioned device |

📸 [Acrobat assigned Required to DYN-Users-Finance](./screenshots/08-acrobat-assigned-required-finance.png)

## Why this assignment is the proof point

This single assignment demonstrates the **user track** delivering payload, and it closes the loop opened in Phase 4's device/user separation:

- The app is targeted at a **user group** (`DYN-Users-Finance`), not a device group. So it follows Aisha to any device she signs into — it is not tied to a particular machine.
- **Install behavior is System**, even though the assignment is user-targeted. Targeting decides *who triggers* the install; install behavior decides *what security context executes* it. Aisha — a Standard user with no local admin — signing in causes a machine-wide install she could not perform herself. Intune acts as the privilege broker: it installs with system rights on behalf of a user who lacks them. This is a clean illustration of least-privilege end-user design coexisting with full application delivery.
- Delivery is via the **Intune Management Extension**, which handles Win32 and Store-new apps. On a freshly provisioned device the IME installs itself first, then processes the app queue — so a user-targeted app typically lands shortly *after* the desktop appears rather than during the Enrollment Status Page. (This is confirmed in the Phase 6 full-flow test, where Acrobat installed post-desktop and was verified both on-device and in the admin-side install report.)

## Assignment types (for reference)

Intune app assignments come in three intents, and choosing correctly is a design decision:

- **Required** — Intune installs the app automatically on in-scope devices/users. Used here: Finance users *get* Acrobat without action.
- **Available for enrolled devices** — the app appears in the Company Portal for the user to install on demand. Appropriate for optional or role-specific software.
- **Uninstall** — Intune actively removes the app from in-scope targets. The mechanism for clawing back software when a user changes department or role.

Because these ride dynamic groups, an attribute change cascades automatically: move a user's department and their app set re-evaluates — Required apps for the new department install, and (if configured with Uninstall assignments for the old set) the previous department's apps can be removed. Access and tooling follow the HR attribute in both directions.

## Extending the matrix

The pattern scales by adding rows: Sales-specific apps assigned to `DYN-Users-Sales`, IT tooling to `DYN-Users-IT`, and organization-wide apps to a broad group or all users. Device-level software (as opposed to per-user apps) would instead target `AP-Devices-All` or a device subset via group tags — the device track rather than the user track. The current single-app matrix is the proven pattern; broadening it is mechanical.

---

[← Back to Phase 5 README](./README.md) | [Dynamic group rules](./dynamic-group-rules.md) | [Project overview](../README.md)

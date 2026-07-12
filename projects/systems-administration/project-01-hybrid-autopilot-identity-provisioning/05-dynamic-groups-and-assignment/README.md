# Phase 5 — Dynamic Groups & Assignment

## Objective

Turn the `department` attribute — set in on-premises AD (Phase 1), synced to Entra (Phase 2) — into the engine that drives cloud access. Department dynamic groups evaluate the attribute automatically, and those groups become the targeting mechanism for application assignment and, later, Conditional Access. This is the **identity-driven** half of the provisioning model: what a user gets is decided by who they are, not configured per device.

This phase is the counterpart to Phase 4. Phase 4 built the **device track** (hardware hash → Autopilot profile → provisioning). Phase 5 builds the **user track** (attribute → dynamic group → app/policy assignment). The two are independent until a user signs in on a provisioned device, at which point they merge — demonstrated end to end in Phase 7.

---

## What was built

### Dynamic user groups

Three department dynamic security groups (`DYN-Users-Finance`, `DYN-Users-Sales`, `DYN-Users-IT`), each matching on `(user.department -eq "…")`, with the attribute-less John Doe account serving as a deliberate negative test that proves membership is genuinely attribute-driven. Full detail, positive verification, and the cause-and-effect negative test: **[dynamic-group-rules.md](./dynamic-group-rules.md)**.

📸 [Dynamic user groups created](./screenshots/01-dynamic-user-groups-created.png) · 📸 [Finance rule](./screenshots/02-finance-dynamic-rule.png) · 📸 [John Doe department blank](./screenshots/06-johndoe-department-blank.png) · 📸 [John Doe in no DYN group](./screenshots/07-johndoe-no-DYN-group-membership.png)

### App assignment

Adobe Acrobat Reader assigned **Required** to `DYN-Users-Finance` — user-targeted, so it follows the user to any device, installed with system rights on behalf of a Standard user (Intune as privilege broker). This assignment is the proof the user track delivers payload, confirmed end to end in the Phase 7 test. Detail and the assignment-type reference: **[app-assignment-matrix.md](./app-assignment-matrix.md)**.

📸 [Acrobat assigned Required to Finance](./screenshots/08-acrobat-assigned-required-finance.png)

### Conditional Access (designed, deferred)

Conditional Access was deliberately **held until after a clean provisioning run**, because CA evaluates during the enrollment sign-in and a mistimed policy would block the very test it needed to survive. The design, the ordering rationale, and the mandatory break-glass exclusion are documented in **[conditional-access-policies.md](./conditional-access-policies.md)**, to be implemented now that Phase 7's clean run is complete.

---

## Design decisions

- **Security groups, Dynamic User membership** — the correct type for app/policy/licensing targeting, and the user-object flavor of dynamic membership (distinct from the device flavor used in Phase 4).
- **Attribute-driven, not manual** — membership follows the HR-sourced `department` value; access follows the data, and data-quality gaps surface as access gaps at the source.
- **User-targeted apps** — Finance's app rides the user group, so it follows the person across devices, complementing the device-targeted provisioning profile.
- **CA deferred by design** — sequencing CA after the baseline provisioning run avoids debugging two systems with identical sign-in-failure symptoms at once.

---

## Outcome

A user's department attribute now drives their group membership and application assignment automatically, with the negative test proving the rules are correctly scoped. Combined with the device provisioning of Phase 4, this completes the identity-driven provisioning chain: one attribute, set once on-premises, flows through sync to determine both what a device becomes and what a user receives on it. Conditional Access — the access-decision capstone that consumes these groups and the device compliance state — is designed and ready to implement.

---

[← Back to Phase 4 — Windows Autopilot](../04-autopilot-hybrid-profile/README.md) | [Project overview](../README.md) | [Continue to Phase 6 — PIM →](../06-pim-for-admin-roles/README.md)

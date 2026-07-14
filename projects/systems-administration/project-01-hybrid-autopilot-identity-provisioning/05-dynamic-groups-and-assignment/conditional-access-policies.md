# Conditional Access Policies — Design (Deferred to Post-Provisioning)

> **Status: designed, not yet implemented.** Conditional Access was deliberately deferred until after a clean end-to-end Autopilot provisioning run (completed in Phase 6). This document records the design and the ordering decision; policy build-out and evidence will be added when the policies are created.

## Why Conditional Access was deferred

Conditional Access policies evaluate during the **sign-in flow** — including the sign-in that happens during Autopilot provisioning. A policy such as "require compliant device" or "require MFA" that fires at the wrong moment can block the very enrollment sign-in the provisioning test depends on, producing a failure whose symptoms ("sign-in failed during OOBE") are indistinguishable from the hybrid-machinery failures the test is actually trying to surface.

The ordering decision was therefore explicit: **build dynamic groups and app assignments before the provisioning test** (they enrich the test — the user's department app arrives during the run) **but hold Conditional Access until after one clean run.** Debugging two systems at once, with identical symptoms, is avoided by sequencing them. This is a deployment-ordering judgment worth stating plainly: CA is powerful and evaluates early, so it is introduced only once the baseline flow is known-good.

With the clean provisioning run now complete (Phase 6), CA can be built against a known-working baseline.

## Planned policies

The intended initial policy set, targeting the same department dynamic groups used for app assignment:

| Policy | Target | Grant control | Rationale |
|---|---|---|---|
| Require MFA for all users | All users (dynamic/all) | Require multi-factor authentication | Baseline identity protection |
| Require compliant or hybrid-joined device | All users | Require device to be marked compliant OR hybrid Entra joined | Ties access to managed devices — the payoff of the hybrid join + Intune enrollment built in Phases 3–4 |
| (Future) Block legacy authentication | All users | Block | Close the legacy-auth bypass of MFA |

## ⚠️ Break-glass exclusion — mandatory

**The break-glass account (`breakglass@...onmicrosoft.com`) MUST be excluded from every Conditional Access policy — especially any MFA requirement — before those policies are enabled.**

This is the single most important CA safety practice and the reason the break-glass account exists (created in Phase 4). Conditional Access misconfigurations are one of the most common causes of tenant-wide admin lockout: a policy that requires MFA or a compliant device, if it catches the only account that could fix it, locks everyone out with no recovery path. The break-glass account — cloud-only Global Admin, excluded from all CA — is the guaranteed way back in.

The exclusion must be in place **as each policy is created**, not added afterward, because a policy is live the moment it is enabled. This carries forward the watch-item flagged in Phase 4.

## Relationship to the rest of the build

Conditional Access is where the identity chain pays its final dividend: because devices are hybrid-joined (Phase 3) and Intune-enrolled with compliance state (Phase 4), and because users are in attribute-driven groups (Phase 5), CA can make access decisions that depend on *both* who the user is and whether their device is managed and compliant. The groups and device states built in the prior phases are the inputs CA consumes. That is why CA comes last — it is the capstone that requires everything else to be in place first.

---

[← Back to Phase 5 README](./README.md) | [Dynamic group rules](./dynamic-group-rules.md) | [Project overview](../README.md)

# Phase 7 — Full-Flow Test & Break/Fix

## Objective

Validate the entire hybrid Autopilot chain end to end by provisioning a real device from a bare OOBE state, with no IT touch, and prove every layer landed: hybrid domain join via the ODJ connector, Entra registration, Intune enrollment, device-to-user mapping, department-driven app delivery, and least-privilege user rights. This phase is where the build stops being a set of configured components and becomes a demonstrated outcome.

It is also, by design, where failures are expected and treated as the most valuable output. Hybrid Autopilot is the most timing- and dependency-sensitive workflow in the project, and the break/fix encountered here — a dormant DNS/DHCP fault dating back to the Phase 1 IP renumber — is the strongest interview artifact the build produced.

---

## The test

A Windows 11 Pro VM (`WIN11-USER01`), registered to Autopilot by hardware hash and with the hybrid deployment profile in the **Assigned** state, was reset to OOBE and provisioned by signing in as a single department user — **Aisha Bello** (`abello@fjlab.ca`, Finance). Nothing else was done to the device; everything below was delivered by the profile chain. The provisioned computer object created by the connector was **`LAB-6KMEnXs2vpf`**.

### Full-flow sequence (successful run)

1. **OOBE — work-or-school sign-in.** The device reached the "Let's set things up for your work or school" screen — the recognition proof: the Autopilot handshake matched the hardware hash and pulled the assigned profile, skipping consumer setup entirely.
   📸 [Work-or-school OOBE sign-in](./screenshots/01-oobe-work-or-school-signin.png)
   📸 [Aisha sign-in](./screenshots/02-oobe-aisha-signin.png)

   *Note:* the sign-in showed generic Microsoft branding rather than tenant branding, because Company Branding was not configured in Entra. This is cosmetic — the profile still drove the flow — but configuring Company Branding is a backlog item for a fully tenant-branded run.

2. **Enrollment Status Page.** The ESP gated the device through three phases — Device preparation, Device setup, Account setup. **Device setup completing is the proof the offline domain join succeeded**: Intune sent the ODJ request to the connector on SYNC01, which created the `LAB-6KMEnXs2vpf` computer object in Lab-Computers and returned the ODJ blob; the device applied it and became domain-joined. During Account setup the flow re-prompted for Aisha's credentials — a normal second authentication step in the hybrid user-driven flow.
   📸 [ESP — Device preparation](./screenshots/03-esp-device-preparation.png)
   📸 [ESP — Account setup re-authentication](./screenshots/04b-esp-account-setup-reauth.png)
   📸 [ESP — Device setup complete, Account setup running](./screenshots/04-esp-device-setup-account-setup.png)

3. **Desktop.** The device reached Aisha's desktop — domain-joined, Intune-enrolled, provisioned.
   📸 [Provisioned desktop](./screenshots/05-provisioned-desktop.png)

### Convergence verification (the three-lists-become-one proof)

Before this run, the device identity existed across separate records that had never converged. After provisioning, one device carries all of them:

- **Hybrid state on the device** — `dsregcmd /status` shows AzureAdJoined YES + DomainJoined YES, achieved by Autopilot rather than the manual Phase 3 path.
  📸 [dsregcmd hybrid state](./screenshots/06-dsregcmd-hybrid-state.png)
- **On-device user-facing view** — Settings → Access work or school shows "corp.lab — Connected to CORP AD domain," the end-user's own view of the hybrid join, complementing the command-line and portal evidence.
  📸 [Device work-or-school / hybrid view](./screenshots/13-device-work-or-school-hybrid.png)
- **On-prem AD object** — `LAB-6KMEnXs2vpf` in Lab-Computers, created by the connector's MSA (the delegation from Phase 4, exercised for real).
  📸 [LAB-6KMEnXs2vpf in Lab-Computers](./screenshots/07-lab-computer-object-aduc.png)
- **Entra device object** — the new hybrid-joined record.
  📸 [Entra hybrid device](./screenshots/08-entra-device-hybrid.png)
- **Intune managed device — Primary user: Aisha Bello.** This single field is the Track 1 / Track 2 merge: the device track (hash → profile → provisioning) and the user track (attributes → groups → assignments) join at the OOBE sign-in, which set Aisha as primary user.
  📸 [Intune managed device, primary user Aisha](./screenshots/09-intune-managed-primary-user-aisha.png)

### Payload verification (the identity-driven provisioning proof)

- **Department app delivered** — Adobe Acrobat Reader, assigned Required to `DYN-Users-Finance`, installed because Aisha (Finance) signed in. Install behavior System, so Intune installed it machine-wide despite Aisha being a Standard user — Intune acting as privilege broker. Confirmed both on-device (Start menu) and admin-side (User install status: 1 install, 0 failures).
  📸 [Acrobat installed — Start menu](./screenshots/10-acrobat-installed.png)
  📸 [Acrobat User install status — Aisha Bello, Installs 1](./screenshots/14-acrobat-installed-for-aisha-bello.png)
- **Least privilege** — Aisha is a Standard user, not a local administrator, per the deployment profile's User account type setting.
  📸 [Aisha not in local administrators](./screenshots/11-aisha-standard-user.png)

---

## Timing

Rough per-phase timings were captured to characterize the flow and seed future comparison. See [full-flow-timing-log.md](./full-flow-timing-log.md).

---

## Break/fix log

The provisioning test surfaced one significant failure — the hybrid post-ODJ domain sign-in — which turned out to have a dormant root cause dating back to the Phase 1 IP renumber. It is the strongest diagnostic case in the project and is documented as a standalone case study.

| # | Symptom | Root cause | Details |
|---|---|---|---|
| Case Study | "Your domain isn't available" at first sign-in after the ODJ reboot | Autopilot-provisioned device booted on DHCP and received DNS from the home router (Rogers gateway at 10.0.0.1), not DC01 — so it could not resolve `corp.lab` to authenticate the domain user. A stale DHCP scope option (leftover `10.10.10.10` from the Phase 1 DC renumber) was a first red herring; the real disease is an unmanaged DHCP layer on the external vSwitch that only surfaced because Autopilot is the first workflow to provision a device that boots on DHCP. | [breakfix-case-study.md](./breakfix-case-study.md) |

A second, minor item worth noting (not a break/fix — nothing broke): the Finance app initially "didn't install" simply because the app object had never been committed — the Store app wizard had been left at the Review + create step without clicking Create. Creating it properly, confirming it appeared in the Windows apps list, and forcing a device sync delivered it. Lesson, consistent with the connector ACL verification and the Case 02 persistence check: the Review + create summary is intent, not confirmation — verify the object actually exists in the list afterward.

---

## Outcome

A device was provisioned end to end with zero touch: hardware hash to fully configured, domain-joined, Intune-managed, department-provisioned, least-privilege desktop — driven entirely by a single user sign-in and the attribute set once in on-premises AD. The one real failure encountered was diagnosed to a root cause two layers removed from its symptom and resolved, with the proper architectural remediation identified. The hybrid Autopilot chain works, and the diagnostic process that got it there is the demonstrable skill.

---

[← Back to Phase 6 — PIM](../06-pim-for-admin-roles/README.md) | [Project overview](../README.md)

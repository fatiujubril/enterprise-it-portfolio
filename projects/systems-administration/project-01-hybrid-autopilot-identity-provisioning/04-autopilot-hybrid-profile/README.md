# Phase 4 — Windows Autopilot (Hybrid Entra Join)

This phase builds the complete Windows Autopilot infrastructure for **user-driven hybrid Entra join**: a sealed device runs OOBE, domain-joins to `corp.lab` without ever pre-touching a domain controller, hybrid-registers to Entra ID, enrolls in Intune, and is gated by an Enrollment Status Page until provisioning completes.

It is also the phase where the identity groundwork from Phases 1–3 gets consumed: the [OU structure](../01-adds-foundation/ou-structure.md) receives the Autopilot computer objects, the [sync scope](../02-entra-connect-sync/sync-scope-config.md) carries the device and user identities to the cloud, and the [hybrid join plumbing](../03-hybrid-join/README.md) is what the Autopilot profile triggers at scale.

---

## How hybrid Autopilot actually works

Cloud-native Autopilot is simple: the device talks to Entra and Intune, both cloud services, done. Hybrid Autopilot has a problem to solve: **Intune cannot create Active Directory computer objects**. It is a cloud service with no domain credentials. The device, meanwhile, is sitting in OOBE with internet access but no domain trust.

The bridge is the **Intune Connector for Active Directory** (also called the ODJ Connector), installed on a domain-joined server. The flow during OOBE:

1. The device authenticates to the Windows Autopilot service, which recognizes its hardware hash and returns the assigned deployment profile.
2. The profile says "hybrid join," so Intune sends an **Offline Domain Join (ODJ)** request to the connector — essentially a work order: *create a computer object with this naming prefix, in this OU, in this domain* (the details come from the Domain Join configuration profile).
3. The connector creates the computer object and generates an ODJ blob (the same artifact `djoin.exe /provision` produces).
4. The blob travels back through Intune to the device, which applies it and reboots into a domain-joined state — having never had line of sight to a DC for the join itself.
5. The user signs in; that first sign-in *does* require DC line of sight, because it is a real Kerberos authentication against `corp.lab`.

The connector communicates outbound-only (long-polling to the Intune service) — no inbound firewall ports on the server.

Two identities travel independently through this phase and only merge at OOBE sign-in:

- **Device track:** hardware hash → Autopilot registration → dynamic device group → deployment profile + domain join config + ESP.
- **User track:** AD account attributes → Entra Connect sync → dynamic user groups → app and policy assignments (Phase 5).

Nobody pre-decides which physical box goes to which employee. Any registered device can be handed to any user; the user's first sign-in is the runtime mapping that personalizes the device. This is what makes the devices interchangeable vessels — the core operational win of Autopilot.

---

## Step 1 — Admin model: scoped roles and break-glass

Before touching enrollment, the tenant's admin model was restructured for least privilege:

- **`cloudadmin@fjlab.ca`** — the daily working admin. Holds Hybrid Identity Administrator, Intune Administrator, License Administrator, and Billing Administrator. Deliberately **not** Global Administrator.

  📸 [cloudadmin scoped role assignments](./screenshots/01-cloudadmin-scoped-roles.png)

- **`breakglass@demolajubriloutlook.onmicrosoft.com`** — cloud-only Global Administrator, reserved for emergencies and rare configuration that genuinely requires GA. Kept on the `.onmicrosoft.com` domain so it survives custom-domain or federation failures.

  📸 [Break-glass account with Global Administrator role](./screenshots/02-breakglass-account-created-with-GA-Role.png)

**Production framing:** this mirrors the standard enterprise pattern — scoped admin roles for daily work, an emergency access account excluded from Conditional Access. ⚠️ **Watch-item carried to Phase 5:** when Conditional Access policies are built, the break-glass account MUST be excluded from all of them (especially MFA requirements) so a CA misconfiguration cannot lock out tenant recovery.

## Step 2 — Licensing: Entra ID P2 + Intune via M365 E5

Hybrid Autopilot needs Intune licensing; the planned Phase 5/6 work (dynamic groups, Conditional Access, PIM) needs Entra ID P1/P2. A standalone EMS E5 trial was not available in this tenant, so the **Microsoft 365 E5 trial** (25 seats, 30-day clock) was activated — E5 bundles both Entra ID P2 and Intune.

📸 [M365 E5 trial activated](./screenshots/03-m365-e5-trial-activated.png)

Aisha Bello (`abello@fjlab.ca`, department Finance) was licensed as the pilot end user.

📸 [E5 license assigned to Aisha](./screenshots/04-aisha-license-assigned.png)

**Lab-pragmatic note:** in production, licensing is procurement, not a build step. In a trial-driven lab it becomes a real dependency — including one non-obvious licensing requirement that surfaced later as this phase's break/fix (see [troubleshooting notes](./troubleshooting-notes.md)).

## Step 3 — MDM authority and automatic enrollment

Two tenant-level switches make enrollment possible:

- **MDM authority = Microsoft Intune** (confirmed already set in this tenant).
- **MDM user scope = All** (Entra ID → Mobility (MDM and WIP) → Microsoft Intune). This is the automatic-enrollment bridge: when a device joins Entra and a licensed, in-scope user is involved, the device continues past the identity layer into MDM enrollment. Without it, Autopilot would hybrid-join the device and simply stop. WIP scope was left at None (Windows Information Protection is deprecated). Enrollment URLs left at defaults.

📸 [MDM user scope set to All](./screenshots/05-mdm-user-scope-all.png)

**Least-privilege reality check:** this control was greyed out for `cloudadmin` — the Intune Administrator role does not include the Entra-level Mobility settings. It was set using the break-glass Global Administrator: a legitimate, logged, one-time elevated-access use, and a concrete preview of the Phase 6 PIM use case (roles that are needed *briefly* should be *eligible*, not permanent).

## Step 4 — Intune Connector for Active Directory

The connector is the hybrid-specific component — the on-prem agent that performs the offline domain join on Intune's behalf — and the most consequential install of this phase. It was installed on SYNC01 as the 2025 **MSA-based** rebuild (the legacy SYSTEM-account connector is no longer supported), enrolled with `cloudadmin`, and scoped so its Managed Service Account (`msaODJtLfqK$`) can create computer objects in **only** the Lab-Computers OU — a delegation configured through the connector's config file *before* MSA setup, and then verified directly in the OU's ACL rather than trusted from the wizard's success dialog.

Full walkthrough — the ODJ mechanics, the 2025 security redesign, account prerequisites, the config-file delegation mechanism and its ordering, and the ACL verification: **[intune-connector-for-ad-setup.md](./intune-connector-for-ad-setup.md)**

📸 [Connector download](./screenshots/06-intune-connector-download.png) · 📸 [Config file OU scoping](./screenshots/07-odj-config-file-lab-computers-ou.png) · 📸 [MSA enrolled](./screenshots/08-connector-msa-enrolled.png) · 📸 [Active in Intune](./screenshots/09-connector-active-in-intune.png) · 📸 [Delegation verified in the OU ACL](./screenshots/10-msa-lab-computers-delegation-verified.png)

The enrollment sign-in initially failed on a non-obvious licensing prerequisite — this phase's break/fix, documented in [troubleshooting-notes.md](./troubleshooting-notes.md).

## Step 5 — Hardware hash capture and import

**Production framing first:** in a real purchase, the OEM or reseller registers devices against the tenant ID at the factory (OEM Direct API / Partner Center) — the vendor never accesses the tenant; the org grants a one-time, revocable registration consent. Manual hash harvesting is the **brownfield** path: existing fleets, refurbished devices, and labs. At scale it is done via Configuration Manager reporting, remote collection (`Get-WindowsAutopilotInfo -Name PC01,PC02,...`), a shared CSV with `-Append`, or skipped entirely with `-Online` (direct Graph registration).

For this lab, the hash was captured on WIN11-USER01 in an elevated PowerShell:

```powershell
Install-Script -Name Get-WindowsAutopilotInfo -Force
Get-WindowsAutopilotInfo -OutputFile C:\Temp\WIN11-USER01-hash.csv
```

The CSV was copied to the Hyper-V host via **PowerShell Direct** — worth knowing because it works over the VMBus with zero guest networking (useful mid-break/fix when a VM's network is deliberately or accidentally broken):

```powershell
$vm = Get-VM | Where-Object Name -like "*WIN11*"
$s = New-PSSession -VMId $vm.VMId -Credential (Get-Credential)
Copy-Item -FromSession $s -Path "C:\Temp\WIN11-USER01-hash.csv" -Destination "C:\Temp\"
Remove-PSSession $s
```

(Gotcha encountered: `New-PSSession -VMName` requires the **Hyper-V VM name**, not the guest computer name — resolving by `VMId` sidesteps the mismatch. Also note `Copy-VMFile` only copies host→guest, not the direction needed here.)

The CSV was imported via Intune → Devices → Enrollment → Windows → Devices → Import.

📸 [Autopilot device imported](./screenshots/11-autopilot-device-imported.png)

The hash uniquely fingerprints the hardware (including TPM data — the VM's vTPM stays enabled). From this point the Autopilot service recognizes this machine at OOBE and hands it the assigned profile instead of consumer setup. **Operational note:** rebuilding the VM (or replacing a motherboard in a physical fleet) changes the hardware identity and requires re-registration.

After import, the same machine exists in three lists that will converge at the provisioning test: Entra devices (hybrid join, Phase 3), Autopilot devices (this import), and Intune managed devices (absent until enrollment). A device identity in Entra does **not** imply Intune management; Intune enrollment always implies (and is anchored to) an Entra device identity.

## Step 6 — Dynamic device group

All Autopilot targeting rides one dynamic **device** group, `AP-Devices-All`, with the standard rule:

```
(device.devicePhysicalIds -any (_ -contains "[ZTDId]"))
```

`devicePhysicalIds` is a multi-valued attribute on the Entra device object; Autopilot registration stamps a `[ZTDId]:...` (Zero Touch Device ID) entry into it. The rule matches every Autopilot-registered device regardless of registration path (CSV, OEM, reseller). The `-any (_ -contains ...)` construction is how dynamic rules iterate multi-valued properties. Dynamic groups are an Entra ID P1 feature — a real licensing dependency, covered here by E5.

📸 [Dynamic membership rule](./screenshots/12-ap-devices-all-dynamic-rule.png)

📸 [WIN11-USER01's Autopilot object evaluated into the group](./screenshots/13-ap-devices-all-win11-member.png)

Dynamic membership evaluates asynchronously — the group is briefly empty after creation; membership was confirmed before assigning anything to it.

**Design note:** one group, multiple payloads. The deployment profile, Domain Join configuration, and ESP are all assigned to this same group, so registering a device into Autopilot automatically wires up everything the device needs. (Group tags — `[OrderID]:...` values matchable in dynamic rules — are the production mechanism for segmenting device *fleets* when device-level differences are needed; department-specific apps remain user-targeted.)

## Step 7 — Deployment profile

`AP-Hybrid-UserDriven`, assigned to `AP-Devices-All`:

📸 [OOBE settings](./screenshots/14-deployment-profile-oobe-settings.png)

| Setting | Value | Why |
|---|---|---|
| Deployment mode | User-Driven | The employee unboxes and signs in themselves — the project's premise. (Self-Deploying is kiosk/shared-device territory and does not support hybrid join.) |
| Join type | Microsoft Entra hybrid joined | The reason this phase exists; this selection is what routes enrollment through the ODJ connector. |
| Skip AD connectivity check | No | OOBE verifies DC reachability before proceeding — a safety net worth keeping on a flat lab network. (Yes exists for VPN-based remote deployment scenarios.) |
| License terms / privacy / account options | Hide | The zero-touch OOBE trim. |
| User account type | **Standard** | The least-privilege moment of this profile: the end user gets a fully provisioned device *without* local admin. |
| Pre-provisioned deployment | No | Technician flow — out of scope for the first run. |
| Convert all targeted devices to Autopilot | No | Would auto-harvest hashes from already-managed devices in the group; powerful for brownfield migration, wrong for a lab with deliberate manual registration. |
| Device name template | *(greyed out)* | For hybrid profiles, naming lives in the Domain Join configuration — it is the on-prem computer object being named, and the connector does the naming. |

Profile assignment to the Autopilot service is asynchronous with three states: **Not assigned → Pending → Assigned**. A device must reach *Assigned* before an OOBE run will pick up the profile — testing while Pending produces a vanilla consumer setup and a confusing afternoon.

📸 [Device showing profile Assigned](./screenshots/15-autopilot-device-profile-assigned.png)

## Step 8 — Domain Join configuration profile

`AP-Hybrid-DomainJoin-corp.lab` (Templates → Domain Join), assigned to the same `AP-Devices-All` group — this small profile is the ODJ connector's work order:

| Setting | Value |
|---|---|
| Computer name prefix | `LAB-` (prefix + random chars must fit the 15-character NetBIOS limit) |
| Domain name | `corp.lab` (full DNS name, not the NetBIOS `CORP`) |
| Organizational unit | `OU=Lab-Computers,OU=Lab-OUs,DC=corp,DC=lab` |

📸 [Domain Join profile summary](./screenshots/16-domain-join-config-profile.png)

⚠️ **The coupling that matters:** the OU here tells Intune *where to ask* the connector to create the object; the connector's config file controls *where the MSA has rights*. These two must stay consistent. If they drift apart, the symptom is an offline-domain-join failure mid-OOBE (the classic 0x80070774 family) — one of the best-known hybrid Autopilot failure modes, with a root cause two layers away from the error message. The profile description documents this dependency in-tenant. Leaving the OU blank falls back to the default Computers container — which "works" but lands objects outside the OU structure and its GPO scoping.

## Step 9 — Enrollment Status Page

The ESP is not cosmetic — it is a **gate**. It blocks the user from reaching a half-configured desktop until required policies and apps have landed, turning "enrollment happened" into "the device is ready for work." Without it, a user could hit the desktop before security baselines apply.

`AP-ESP-Standard`, assigned to `AP-Devices-All`:

- Show progress: **Yes**; error timeout **60 minutes** (hybrid runs are slower — the ODJ round-trip and first domain contact add time; do not shorten this).
- Log collection / diagnostics for end users: **Yes** — in the lab this serves the admin: a failed OOBE run offers diagnostics on the error screen instead of a blind retry.
- Only show for OOBE-provisioned devices: **Yes**.
- **Block device use until all apps and profiles are installed: Yes**, scope **All** — the gate itself.
- Allow reset / allow use on error: **No** — failures should stop and show themselves, not be clicked past.
- Install Windows updates during OOBE: **No for the first run** — a deliberate variable-reduction decision. Hybrid is already the most timing-fragile Autopilot flavor; the first test should have a suspect list of one (the hybrid machinery), not "an update took 40 minutes and tripped the ESP timer." The production-appropriate setting is Yes, to be enabled after one clean end-to-end run.

📸 [ESP profile summary](./screenshots/17-esp-profile.png)

The tenant also carries a built-in **Default** ESP that cannot be deleted, only edited. Priority decides: the custom profile at priority 1 wins for devices in `AP-Devices-All`; unmatched devices fall through to Default (which ships with progress display off). Production tenants use this priority stack deliberately — strict ESP for corporate Autopilot devices, lighter fallback for other enrollments.

📸 [ESP priority list — custom profile above the immutable Default](./screenshots/18-esp-priority-list.png)

---

## Build state and what comes next

Complete chain, in dependency order:

**Connector (MSA, OU-scoped) → hardware hash registered → dynamic device group → deployment profile + Domain Join config + ESP, all riding one group.**

The remaining Phase 4 milestone is the provisioning test itself: reset WIN11-USER01, run OOBE, and watch the device domain-join, hybrid-register, enroll, and gate through ESP with zero IT touch. Hybrid Autopilot is the most failure-prone phase of this architecture (Intune Connector behavior, ODJ timing, DC line-of-sight during OOBE, profile/sync timing) — failures during the test are expected and will be documented as break/fix material, with the full reset-and-provision analysis in Phase 7.

Known watch-item already banked for Phase 5: build dynamic user groups and app assignments **before** the test (richer ESP payload), but hold Conditional Access until after one clean run — CA evaluates during the enrollment sign-in flow, and a badly-timed policy produces symptoms identical to hybrid-machinery failures.

---

## Break/fix log

| # | Symptom | Root cause | Details |
|---|---|---|---|
| breakfix-01 | Connector wizard sign-in fails with generic "Something went wrong — An unanticipated error occurred" | Enrolling admin met the *role* prerequisite but not the *license* prerequisite (Intune license missing / still propagating) | [troubleshooting-notes.md](./troubleshooting-notes.md) |

---

[← Back to Phase 3](../03-hybrid-join/README.md) | [Project overview](../README.md) | [Continue to Phase 5 →](../05-dynamic-groups-and-assignment/README.md)

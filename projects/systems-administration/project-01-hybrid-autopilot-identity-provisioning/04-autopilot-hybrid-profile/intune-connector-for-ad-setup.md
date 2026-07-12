# Intune Connector for Active Directory — Setup and Delegation

The connector is the component that makes hybrid Autopilot possible, and the most consequential install of this phase. This document covers what it does, the 2025 security redesign, the install on SYNC01, and — the part wizards don't show — verifying that the delegation it claims to create actually exists in AD.

---

## What the connector does

Intune cannot create Active Directory computer objects — it is a cloud service with no domain credentials. During a hybrid Autopilot OOBE run, the device has internet access but no domain trust. The connector bridges the gap:

1. Intune receives the enrolling device, sees a hybrid-join deployment profile, and sends the connector an **Offline Domain Join (ODJ)** request — a work order built from the Domain Join configuration profile: *create a computer object with this prefix, in this OU, in this domain*.
2. The connector (running on a domain-joined server) creates the computer object and generates an ODJ blob — the same artifact `djoin.exe /provision` produces.
3. The blob returns through Intune to the device, which applies it and reboots domain-joined, having never had DC line-of-sight for the join itself. (The user's first sign-in *does* need DC reachability — that is a real Kerberos authentication.)

Communication is outbound-only long-polling to the Intune service: no inbound firewall ports on the server.

## The 2025 security redesign — MSA instead of SYSTEM

The connector installed here is **not** the one most older guides describe. As part of Microsoft's Secure Future Initiative, it was rebuilt in 2025 to run under a **Managed Service Account (MSA)** instead of the local SYSTEM account. The legacy SYSTEM-based connector lost support in June 2025, and versions older than 6.2501.2000.5 can no longer process enrollment requests.

Why it matters: under the old model, the OU delegation was granted to the *server's computer account*, and the service ran as SYSTEM — so compromising the server meant inheriting the delegation wholesale. The MSA model is least privilege applied to the connector itself:

- automatic password rotation handled by AD,
- bound to a single domain-joined machine,
- holding create-computer rights **only** on explicitly listed OUs.

A useful side effect of the rebuild: from version 6.2504.2001.8 the wizard renders sign-in through WebView2 (Edge-based), so the old "disable IE Enhanced Security Configuration first" step in every pre-2025 guide is obsolete.

## Accounts and prerequisites

The connector was installed on SYNC01 — already a domain member running Entra Connect. **Production framing:** these would typically be separate servers, and a second connector on another server is the high-availability pattern; a single co-located connector is a lab-pragmatic single point of failure.

Two accounts are involved, with different requirements:

| Account | Requirements |
|---|---|
| Domain account running the installer | Local admin on the server; rights to create `msDS-ManagedServiceAccount` objects in the Managed Service Accounts container; (optional but smoother) rights to modify OU permissions, so the wizard can set the MSA's OU delegation itself |
| Entra account signing in to enroll | **Intune Administrator role AND an assigned Intune license** |

The license half of the enrollment requirement is non-obvious — admin accounts are often deliberately unlicensed — and missing it produced this phase's break/fix: the wizard fails with a generic "Something went wrong" that names neither the check nor the fix. Full writeup: [troubleshooting-notes.md](./troubleshooting-notes.md). The enrollment account is a point-in-time requirement only; the license can be reclaimed after enrollment.

## Install sequence — and why the order matters

📸 [Connector download from the Intune admin center](./screenshots/06-intune-connector-download.png)

1. Downloaded from Intune admin center → Devices → Enrollment → Windows → Intune Connector for Active Directory, and ran `ODJConnectorBootstrapper.exe` on SYNC01.
2. At the post-install "Configure now / Close" prompt: **Close** — deliberately. See next section.
3. Edited the connector config file **before** enrollment.
4. Launched the enrollment wizard manually, signed in with `cloudadmin@fjlab.ca` (after resolving the license break/fix), and ran **Configure Managed Service Account**.

The enrollment auto-generated the MSA `msaODJtLfqK$` (names follow the pattern `msaODJ#####`; the `$` suffix marks a service-type account — note the name down, it is the principal you verify delegation against).

📸 [Connector enrolled, MSA created](./screenshots/08-connector-msa-enrolled.png)

📸 [Connector showing Active in the Intune admin center](./screenshots/09-connector-active-in-intune.png)

## OU scoping via the config file — the delegation mechanism

By default the MSA only receives rights on the domain's default **Computers** container. This lab targets a custom OU, so the DN was added to `ODJConnectorEnrollmentWizard.exe.config` (in `C:\Program Files\Microsoft Intune\ODJConnector\ODJConnectorEnrollmentWizard\`, edited in an elevated Notepad since Program Files is protected):

```xml
<add key="OrganizationalUnitsUsedForOfflineDomainJoin"
     value="OU=Lab-Computers,OU=Lab-OUs,DC=corp,DC=lab" />
```

📸 [Config file with Lab-Computers DN in place](./screenshots/07-odj-config-file-lab-computers-ou.png)

This is a design detail, not a click-path detail: **the config file IS the delegation mechanism.** When "Configure Managed Service Account" runs, the MSA is granted create-computer rights on exactly the OUs listed in the file (plus the default Computers container). Hence the ordering above — edit first, configure once, one clean pass. Configuring first with the default file would have granted rights only on the default container and required a re-run after the edit. Removing an OU from the file and re-running *revokes* that OU's access. (Multiple OUs would be semicolon-separated in the same value.)

⚠️ **Coupling watch-item:** this OU list must stay consistent with the Organizational Unit in the Domain Join configuration profile — Intune's side says *where to ask*, the config file controls *where the MSA has rights*. If they drift, the symptom is an ODJ failure mid-OOBE (the 0x80070774 family) with a root cause two layers away from the error message. The dependency is documented in the Domain Join profile's description in-tenant.

## Verifying the delegation actually landed

Wizard success dialogs are claims; the AD ACL is the proof. In ADUC on DC01 (View → Advanced Features), Lab-Computers → Properties → Security → Advanced:

📸 [MSA create-computer delegation on Lab-Computers](./screenshots/10-msa-lab-computers-delegation-verified.png)

Reading the full ACL — no unexplained entries policy:

- `Allow — msaODJtLfqK$ — Create Computer objects — This object only`: the connector's grant. "This object only" is correct, not a limitation — create-child rights live on the *container*, so this means "may create computer objects directly inside Lab-Computers," and nothing else, nowhere else. The connector cannot touch Lab-Servers, Lab-Users, or any other OU. The MSA also appears in the Managed Service Accounts container at the domain root.
- `Deny — Everyone — Delete / Delete subtree`: **not** connector-related. This is the standard "Protect object from accidental deletion" checkbox (ticked by default at OU creation), implemented as a deny ACE. Deny wins over allow in AD ACL evaluation, so even a Domain Admin must consciously untick the protection flag before deleting the OU — a one-click accident becomes a two-step deliberate act.

**The end-to-end least-privilege chain:** a cloud-side enrollment produced an on-prem delegation scoped to a single OU, held by an auto-rotating service account bound to one server, verifiable in one ACL entry.

---

[← Back to Phase 4 README](./README.md) | [Project overview](../README.md)

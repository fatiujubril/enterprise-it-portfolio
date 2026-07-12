# Phase 3 — Hybrid Azure AD Join

## Objective

Configure hybrid Azure AD join so that domain-joined Windows devices automatically register to Microsoft Entra ID, giving each device a dual identity: domain-joined on-premises and registered in the cloud. This dual state is what allows a device to retain on-premises management (Group Policy, Kerberos, on-prem resource access) while also being recognized and trusted by Entra ID, Intune, and Conditional Access.

This is the device-side foundation that Phase 4 (Autopilot) and Phase 5 (dynamic group provisioning) build upon.

---

## How Hybrid Join Works

A hybrid-joined device holds two simultaneous identities:

- **Domain-joined** to on-premises AD (`corp.lab`) — the traditional join.
- **Registered in Entra ID** — a cloud device object that lets the cloud recognize and trust the device.

The mechanism that connects them:

1. **The Service Connection Point (SCP)** is an object written into AD's configuration partition that tells domain-joined devices which Entra tenant to register with. Without it, a domain-joined device has no knowledge of the cloud tenant. Entra Connect writes the SCP.
2. A scheduled task on each domain-joined device (`Automatic-Device-Join`) reads the SCP, discovers the tenant, and registers the device to Entra ID. It runs on startup, after sign-in, and on a periodic timer — so registration is eventually automatic.
3. Registration requires the device to reach both a **domain controller** (to read the SCP and authenticate) and the **internet** (to reach the Entra device registration endpoint). This dual requirement is the most common real-world failure point.
4. Once registered, the device appears in Entra ID as **Microsoft Entra hybrid joined**.

**Timing dependency:** hybrid join is not instant. Because it depends on the scheduled task firing and Entra Connect's sync cycle, a device may show as not-yet-joined for a period. This is convergence, not failure — the correct response is to check state with `dsregcmd /status` rather than assume something is broken.

---

## Build Steps

### 1. Configure device options in Entra Connect (write the SCP)

The Entra Connect wizard was re-run on SYNC01 and the **Configure device options** → **Configure Hybrid Azure AD join** task selected. Device systems were set to **Windows 10 or later domain-joined devices** (the modern, token-based path).

📸 [Configure device options task](./screenshots/01-connect-device-options.png) · 📸 [Hybrid Azure AD join selected](./screenshots/02-hybrid-join-selected.png)

The SCP was configured for the `corp.lab` forest, with **Authentication Service set to Microsoft Entra ID** — the modern cloud-auth path, not legacy AD FS federation. This is the correct choice for a [Password Hash Synchronization](../02-entra-connect-sync/auth-method-decision.md) environment: 📸 [SCP configuration](./screenshots/03-scp-config-forest.png)

Configuration completed successfully: 📸 [Device options complete](./screenshots/04-device-options-complete.png)

### 2. Verify the SCP was written

The SCP was confirmed present in AD, pointing at the correct tenant — showing both the tenant name and the tenant GUID (`382f79d8-...`), which matches the tenant ID. The SCP references the tenant's canonical `.onmicrosoft.com` identity, which is expected regardless of custom domains added: 📸 [SCP verified](./screenshots/05-scp-verified.png)

### 3. Join and register the client (WIN11-USER01)

A Windows 11 Pro client (`WIN11-USER01`) was domain-joined to `corp.lab`. Windows 11 Pro is required — Home edition cannot domain-join or hybrid-join. The VM has a virtual TPM enabled, which protects the device's cryptographic keys (the Primary Refresh Token and device certificate are TPM-bound) and is also used for Autopilot attestation in Phase 4.

The client was confirmed domain-joined, with DNS pointing at DC01 (`10.0.0.5`) so it can resolve `corp.lab` — the line-of-sight prerequisite for both authentication and device registration: 📸 [Client domain-joined](./screenshots/06-client-domain-joined.png) · 📸 [Client DNS configuration](./screenshots/07-client-dns-config.png)

A synced department user (**Aisha Bello**, `abello@fjlab.ca`, Finance) was signed in — a real synced identity rather than a local account, so the device registers under a genuine directory user and is positioned for department-based provisioning in Phase 5.

Immediately after sign-in, the device showed the expected pre-registration state — domain-joined but not yet Entra-registered: 📸 [dsregcmd before — AzureAdJoined NO](./screenshots/08-dsregcmd-before.png)

The `Automatic-Device-Join` scheduled task was triggered to accelerate registration (in production this happens automatically on the task's normal schedule). After it ran, the device reached full hybrid-joined state — both AzureAdJoined and DomainJoined YES: 📸 [dsregcmd after — AzureAdJoined YES](./screenshots/09-dsregcmd-after.png)

### 4. Verify in the cloud

The device appeared in Entra ID as **Microsoft Entra hybrid joined**, with a Device ID that cross-references the `dsregcmd` output on the client — proving the on-premises device and the cloud object are the same entity: 📸 [Device hybrid joined in Entra](./screenshots/10-entra-device-hybrid-joined.png)

---

## Hardening — Excluding Servers and Domain Controllers

Enabling hybrid join surfaced an unintended consequence that required a hardening fix. Because hybrid join is configured domain-wide via the SCP, **every** domain-joined Windows machine — including the domain controller and the sync server — attempted to register to Entra ID. This is documented in full in [Case 01](./troubleshooting-case-01-servers-registered.md), and resolved by scoping device registration to client OUs only via Group Policy, following Microsoft's guidance that domain controllers must never be device-registered.

The remediation had a sequel: days later the server records **reappeared** in Entra — not because the GPO failed, but because the original cleanup missed a third place where registration state lives (the `userCertificate` attribute on the AD computer objects, which the sync engine re-exported). The differential diagnosis and completed fix are documented in [Case 02](./troubleshooting-case-02-usercertificate-resync.md).

The end state: only the intended client (`WIN11-USER01`) holds a cloud device identity; DC01 and SYNC01 remain plain domain members — verified to persist across sync cycles.

---

## Outcome

A Windows 11 client is now hybrid Azure AD joined — domain-joined on-premises and registered in Entra ID under a real synced user — while infrastructure servers are correctly excluded from device registration. The device now has the cloud identity required for Intune enrollment, Autopilot, and Conditional Access in the phases ahead.

---

## Break/fix log

| # | Symptom | Root cause | Details |
|---|---|---|---|
| Case 01 | DC01 and SYNC01 unintentionally registered as Entra devices after hybrid join was enabled | SCP is forest-wide and `Automatic-Device-Join` runs on every domain-joined machine, servers included; registration needed GPO scoping (including a disabled-link diagnostic) | [troubleshooting-case-01-servers-registered.md](./troubleshooting-case-01-servers-registered.md) |
| Case 02 | The same server records reappeared in Entra days after remediation | Original cleanup missed the third place registration state lives — the stale `userCertificate` AD attribute, re-exported by the sync engine | [troubleshooting-case-02-usercertificate-resync.md](./troubleshooting-case-02-usercertificate-resync.md) |

---

[← Back to Phase 2 — Entra Connect Sync](../02-entra-connect-sync/README.md) | [Project overview](../README.md) | [Continue to Phase 4 — Windows Autopilot →](../04-autopilot-hybrid-profile/README.md)

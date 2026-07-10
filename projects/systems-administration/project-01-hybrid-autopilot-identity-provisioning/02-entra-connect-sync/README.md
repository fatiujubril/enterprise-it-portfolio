# Phase 2 — Entra Connect Directory Synchronization

## Objective

Establish directory synchronization between the on-premises `corp.lab` Active Directory and Microsoft Entra ID, so that on-premises identities exist in the cloud with clean, routable identities. This is the foundation of hybrid identity: the on-premises directory remains the source of truth, and Entra Connect projects those identities into the cloud where they can drive Conditional Access, dynamic group membership, and eventually hybrid device join.

This phase has two halves: preparing routable identities (UPN suffix remediation) and deploying Entra Connect on a dedicated member server.

---

## Part 1 — Routable Identity Preparation

### The problem

The on-premises AD domain is `corp.lab`, a non-routable domain. Entra ID only accepts a UPN suffix that matches a **verified** domain in the tenant. Left unaddressed, synced users would either fall back to the `.onmicrosoft.com` suffix or fail sign-in — the classic "synced users cannot log in to Microsoft 365" scenario. The Entra Connect installer itself flags this before installation.

### The fix

A routable public domain, `fjlab.ca`, was registered and verified in the tenant, then added as an alternative UPN suffix in Active Directory. Each test user's UPN suffix was switched from `@corp.lab` to `@fjlab.ca` so their on-premises UPN matches a verified cloud domain.

**Sequence matters:** the domain was verified in Entra *before* changing on-premises UPNs. Verifying cloud-side first means that when sync runs, the UPNs carry through cleanly with no fallback.

`fjlab.ca` verified in the tenant alongside the default `.onmicrosoft.com` domain: 📸 [Entra custom domain verified](./screenshots/01-entra-domain-verified.png)

Verification was completed by adding a TXT record at the domain registrar: 📸 [GoDaddy TXT record](./screenshots/02-godaddy-txt-record.png)

`fjlab.ca` added as an alternative UPN suffix in AD Domains and Trusts: 📸 [UPN suffix added](./screenshots/03-upn-suffix-added.png)

Each user's logon suffix switched to the routable domain: 📸 [User account UPN changed](./screenshots/04-user-account-upn-changed.png)

All four users confirmed on the `@fjlab.ca` suffix: 📸 [UPN verification table](./screenshots/05-upn-verification-table.png)

---

## Part 2 — Entra Connect Deployment

### Pre-flight decisions

Four decisions were made deliberately before installation, each documented with rationale in the supporting files:

| Decision | Choice | Rationale |
|---|---|---|
| Install location | Dedicated member server (`SYNC01`) | Keeps sync workload off the domain controller (production topology) |
| Authentication method | Password Hash Synchronization | Modern default; resilient to on-prem outage; enables cloud security features. See `auth-method-decision.md` |
| Sync scope | `Lab-OUs` only, excluding `Lab-Admins` | Sync only what needs cloud presence; keep privileged accounts on-prem. See `sync-scope-config.md` |
| Install path | Custom | The only path exposing OU filtering and the auth-method choice |

### Member server build (SYNC01)

Rather than installing Entra Connect on the domain controller, a dedicated member server was provisioned — the production-recommended topology, which keeps the sync service, its database, and its privileged connector account off the most sensitive box in the environment.

`SYNC01` was given a static IP (`10.0.0.6`) with DNS pointed at DC01 (`10.0.0.5`) — the critical prerequisite, since only the DC can resolve `corp.lab` for domain discovery: 📸 [SYNC01 network configuration](./screenshots/06-sync01-network-config.png)

Domain discovery confirmed DC01 before joining: 📸 [SYNC01 domain discovery](./screenshots/07-sync01-domain-discovery.png)

Domain join verified, with `Test-ComputerSecureChannel` returning True: 📸 [SYNC01 domain joined](./screenshots/08-sync01-domain-joined.png)

Outbound connectivity to Microsoft endpoints confirmed on port 443 before installing — a deliberate prerequisite check rather than installing blind: 📸 [Connectivity verified](./screenshots/09-sync01-connectivity-verified.png)

### Installation

The installer's Express path was declined in favour of Custom. The installer itself flagged the non-routable domain and recommended custom settings — validating the Part 1 preparation: 📸 [Express vs Custom warning](./screenshots/10-entra-connect-express-vs-custom-warning.png)

**Password Hash Synchronization** was selected as the sign-in method. Seamless SSO was deliberately left disabled, because hybrid-joined devices (the project's end goal) obtain SSO natively through the Primary Refresh Token, making Seamless SSO redundant for this design: 📸 [PHS selected](./screenshots/11-user-signin-phs-selected.png)

A **dedicated cloud admin** account (`cloudadmin@fjlab.ca`) was created with the **Hybrid Identity Administrator** role rather than using the personal Microsoft account that created the tenant, or a full Global Administrator. Hybrid Identity Administrator is the least-privilege role sufficient for Entra Connect setup: 📸 [Dedicated cloud admin with least-privilege role](./screenshots/12-dedicated-cloud-admin-hybrid-identity-role.png)

The on-premises directory was connected. Rather than running sync as Domain Admin, the wizard was allowed to create a dedicated `MSOL_` connector account scoped with least privilege for the sync task — the on-premises mirror of the least-privilege cloud role: 📸 [AD directory connected](./screenshots/13-connect-ad-account-creation.png)

The Entra sign-in configuration confirmed `fjlab.ca` as **Verified** with `userPrincipalName` as the source attribute — the payoff of the Part 1 UPN work. The unverified `corp.lab` suffix is intentionally not used for user sign-in, so the continue-anyway option was correctly acknowledged: 📸 [UPN suffix verified in wizard](./screenshots/14-entra-signin-upn-verified.png)

**OU filtering** — the most important scoping decision. Sync was restricted to `Lab-Computers`, `Lab-Servers`, and `Lab-Users`, with `Lab-Admins` deliberately excluded so privileged on-premises identities have no cloud footprint. All built-in containers (service accounts, system objects) were excluded: 📸 [OU filtering scoped](./screenshots/15-ou-filtering-scoped.png)

The source anchor was left as Azure-managed (`ms-DS-ConsistencyGuid`) — the writable, migration-resilient attribute that permanently links each on-premises object to its cloud counterpart: 📸 [Source anchor configuration](./screenshots/16-source-anchor-config.png)

Optional features were left at defaults (PHS only). **Device writeback was deliberately not enabled** — a commonly misunderstood setting. Device writeback writes cloud devices *down* to on-prem for AD FS scenarios; it is unrelated to hybrid join, which is configured separately in Phase 3 and goes the opposite direction: 📸 [Optional features defaults](./screenshots/17-optional-features-defaults.png)

The final configuration summary confirmed both connectors, PHS, source anchor, and sync-on-completion, with staging mode correctly disabled: 📸 [Ready to configure summary](./screenshots/18-ready-to-configure-summary.png)

Configuration completed successfully. Post-install advisories were reviewed: the AD Recycle Bin recommendation was actioned on DC01, TPM was skipped as unnecessary for a lab VM, and the Connect Health Agent limitation was noted as expected on the Free tier (Health monitoring requires P1/P2, deferred to the EMS E5 trial in Phase 4): 📸 [Configuration complete](./screenshots/19-configuration-complete.png)

---

## Verification

The sync scheduler confirmed active (not staging) with a healthy 30-minute delta cycle. All four on-premises users appeared in Entra ID with clean `@fjlab.ca` identities: 📸 [Synced users in Entra ID](./screenshots/20-entra-users-synced.png)

Attribute-level verification on a synced user confirmed the sync worked end to end: **On-premises sync enabled: Yes**, **Department: Finance** synced correctly (the attribute that drives dynamic groups in Phase 5), and the `ms-DS-ConsistencyGuid` source anchor present as the immutable link: 📸 [User sync source and attributes](./screenshots/21-user-sync-source-and-attributes.png)

User sign-in and app access were deferred pending licensing — synced users have no license on the Free tier, so app access is unavailable until the EMS E5 trial is activated in Phase 4. Sync itself is fully proven via directory attributes.

---

## Security Decisions Summary

This phase applied least-privilege consistently across three layers:

- **Cloud admin role** — Hybrid Identity Administrator, not Global Administrator
- **On-premises sync account** — dedicated `MSOL_` connector account, not Domain Admin
- **Sync scope** — privileged `Lab-Admins` OU excluded from cloud sync

Together these demonstrate that identity synchronization is not just a mechanical "run the wizard" task but a series of deliberate security decisions about what should exist in the cloud and with what privilege.

---

## Outcome

On-premises identities are now synchronized to Entra ID with routable `@fjlab.ca` UPNs and correct department attributes, on a production-style topology with least-privilege accounts throughout. This synchronized identity foundation is what Phase 3 (hybrid Azure AD join) and Phase 5 (dynamic group-driven provisioning) build upon.

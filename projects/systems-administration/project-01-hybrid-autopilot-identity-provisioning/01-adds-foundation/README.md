# Phase 1 — On-Premises Active Directory Foundation

## Objective

Establish the on-premises identity foundation that the rest of the hybrid build depends on: a domain controller running AD DS, a tiered organizational unit structure, and department-attributed user accounts. In a hybrid identity model, on-premises Active Directory is the **source of truth** — identities originate here and are later synchronized up to Entra ID. Every downstream capability (Entra Connect sync, hybrid join, dynamic group membership, app assignment) ultimately keys off attributes set at this layer.

---

## Environment

| Component | Value |
|---|---|
| Server role | Domain Controller (AD DS, DNS, DHCP) |
| Hostname | DC01 |
| Domain | corp.lab |
| Static IP | 10.0.0.5 /24 |
| Gateway | 10.0.0.1 |
| DNS (self-reference) | 10.0.0.5 |
| DNS forwarders | 8.8.8.8, 8.8.4.4 |
| OS | Windows Server 2022 |

The domain controller runs as a virtual machine on a Hyper-V host. The host is the persistent environment; VMs are treated as disposable lab infrastructure that can be rebuilt without losing project documentation.

---

## Design Decisions

**Domain name — `corp.lab`.** The internal AD domain name is kept as `corp.lab`. While a non-routable suffix like this creates a UPN-matching consideration for hybrid identity (addressed in Phase 2 by adding a routable `fjlab.ca` UPN suffix), it is retained as the AD domain name because renaming a domain is a disruptive operation with no benefit here. The AD domain and the UPN suffix used for cloud sign-in are two separate things.

**Static IP and self-referencing DNS.** A domain controller must have a static IP and must point its DNS at itself, because the DC *is* the authoritative DNS server for the domain. Internet name resolution is handled through DNS forwarders configured inside the DNS console, not by placing public DNS servers on the network adapter. This distinction is important and is the root of the break/fix documented in `troubleshooting-notes.md`.

**Tiered OU structure.** Organizational units are separated by function — `Lab-Admins`, `Lab-Computers`, `Lab-Servers`, `Lab-Users` — nested under a parent `Lab-OUs`. This separation supports scoped sync filtering in Phase 2 (only the required OUs are synchronized) and reflects an administrative-tiering approach rather than a flat directory.

---

## Build Steps

1. Provisioned Windows Server 2022 VM, assigned a static IP, and renamed the host to `DC01` prior to promotion.
2. Installed the AD DS role and promoted the server to a domain controller for a new forest, `corp.lab`, with DNS installed during promotion. 📸 [Server Manager — AD DS, DNS, and DHCP roles on DC01 (static IP 10.0.0.5)](./screenshots/01-server-manager-adds-role.png)
3. Configured the network adapter with a static IP and self-referencing DNS, plus DNS forwarders for external resolution. 📸 [Static IP and self-referencing DNS](./screenshots/06-static-ip-dns-config.png) · 📸 [DNS forwarders (8.8.8.8 / 8.8.4.4)](./screenshots/07-dns-forwarders.png)
4. Created the `Lab-OUs` tiered organizational unit structure. 📸 [corp.lab directory tree](./screenshots/02-domain-overview-aduc.png) · 📸 [Tiered OU hierarchy](./screenshots/03-ou-structure.png)
5. Created department-attributed test user accounts to serve as the identities that will later drive dynamic group membership.

---

## Test Users

Three department-attributed users were created, plus one intentionally attribute-less account:

| Name | SamAccountName | Department | Purpose |
|---|---|---|---|
| Aisha Bello | abello | Finance | Dynamic group membership test |
| Marcus Chen | mchen | Sales | Dynamic group membership test |
| Priya Sharma | psharma | IT | Dynamic group membership test |
| John Doe | j.doe | *(none)* | Negative test — should match no dynamic group |

The `department` attribute is the value Entra Connect synchronizes and that dynamic group rules evaluate in Phase 5. John Doe is deliberately left without a department to validate that dynamic membership rules are scoped correctly and do not inadvertently capture every user.

📸 [Test users created with department attributes](./screenshots/04-test-users-created.png) · 📸 [User Organization tab confirming Department field populated](./screenshots/05-user-department-attribute.png)

---

## Verification

Confirmed via PowerShell that all users exist in the target OU with correct department attributes:

```powershell
Get-ADUser -Filter * -SearchBase "OU=Lab-Users,OU=Lab-OUs,DC=corp,DC=lab" -Properties Department |
    Select-Object Name, SamAccountName, UserPrincipalName, Department | Format-Table -AutoSize
```

---

## Outcome

A healthy domain controller with a clean tiered directory and department-attributed identities, verified via `dcdiag`. This is the on-premises source of truth that Phase 2 synchronizes to Entra ID.

---

[← Project overview](../README.md) | [Continue to Phase 2 — Entra Connect Sync →](../02-entra-connect-sync/README.md)

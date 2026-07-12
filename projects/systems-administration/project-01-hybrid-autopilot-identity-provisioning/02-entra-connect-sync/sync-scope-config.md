# Sync Scope Configuration — OU Filtering

## The decision

Directory synchronization was scoped to specific organizational units rather than syncing the entire directory. Within the `Lab-OUs` structure, three of the four child OUs were synced and one — `Lab-Admins` — was deliberately excluded.

## What is synced

| OU | Synced | Reason |
|---|---|---|
| `Lab-Users` | Yes | Contains the department-attributed test users that need cloud identities |
| `Lab-Computers` | Yes | Target OU for hybrid-joined device objects in Phase 4 |
| `Lab-Servers` | Yes | Member servers; low sensitivity, included for completeness |
| `Lab-Admins` | **No** | Privileged accounts deliberately kept out of the cloud |
| All built-in containers | No | Service accounts, system objects, and default containers have no business in the cloud |

## Why filter at all

Syncing an entire Active Directory to the cloud is neither necessary nor safe. Built-in containers hold service accounts, disabled objects, computer objects, and system accounts — syncing them is noise at best and expands the cloud attack surface at worst. Scoping sync to only the objects that require a cloud presence is a deliberate security decision, not a convenience.

In production, this filtering is typically preceded by directory cleanup (for example, running **IdFix** to identify objects that would fail sync — duplicate proxy addresses, invalid UPN characters, blank required attributes). The lab directory is clean, so this step was not required, but the principle — clean and scope before syncing — is the same.

## Why exclude Lab-Admins specifically

Privileged on-premises identities should not have a cloud footprint unless there is a specific need for one. Excluding the admin OU from sync means:

- Administrative accounts cannot be targeted through cloud-based attack paths (cloud phishing, password spray against the cloud sign-in endpoint).
- The identity attack surface in Entra ID is reduced to only standard user accounts.
- It applies least-privilege thinking to sync *scope*, consistent with the least-privilege choices made for the cloud admin role (Hybrid Identity Administrator) and the on-premises connector account (dedicated `MSOL_` account).

This is the kind of decision that distinguishes a deliberate hybrid identity design from a default "sync everything" deployment.

## Relationship to the second filtering layer

Entra Connect offers two independent filtering mechanisms that stack:

1. **OU filtering** (used here) — the broad structural boundary defining which containers sync.
2. **Group-based filtering** (not used here) — an optional additional filter restricting sync to members of a single named group, typically used for controlled pilot waves within an already-scoped OU set.

With only four users, OU filtering alone provides the right level of control. In a large production rollout, both layers might be used together — OU filtering for structure, group filtering for a phased pilot.

## Note on privileged identity synchronization

Excluding admin accounts from sync is one approach to protecting privileged identities in a hybrid environment. A complementary or alternative approach in larger environments is to keep cloud administrative identities entirely cloud-native (created directly in Entra ID, never synced from on-prem), so that a compromise of on-premises AD cannot directly yield cloud administrative access. The `cloudadmin@fjlab.ca` account in this build is exactly such a cloud-native admin — created in the cloud, not synced from `corp.lab`.

---

[← Back to Phase 2 README](./README.md) | [Project overview](../README.md)

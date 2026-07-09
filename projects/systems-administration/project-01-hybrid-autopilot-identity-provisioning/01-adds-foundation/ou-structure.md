# OU Structure

## Layout

```
corp.lab
└── Lab-OUs
    ├── Lab-Admins        Administrative accounts (tier separation)
    ├── Lab-Computers     Target OU for hybrid-joined device objects (Phase 4)
    ├── Lab-Servers       Member servers
    └── Lab-Users         Standard user accounts (department-attributed)
```

## Rationale

**Why a parent `Lab-OUs` container.** Grouping all custom OUs under a single parent keeps lab-created objects separate from the built-in containers (`Users`, `Computers`, `Domain Controllers`) that ship with Active Directory. This makes sync scoping in Phase 2 straightforward — the sync boundary can be drawn cleanly around `Lab-OUs` without accidentally including built-in service or system accounts.

**Why separate Users / Computers / Servers / Admins.** Function-based separation is the foundation of administrative tiering. It allows:

- **Scoped synchronization** — only the OUs containing identities that need to exist in the cloud are synced. Admin and server accounts can be deliberately excluded.
- **Targeted Group Policy** — policy can be applied per OU rather than domain-wide.
- **Least-privilege delegation** — administrative rights over one OU can be granted without granting rights over the entire domain.

**Role of `Lab-Computers` in later phases.** This OU becomes the destination where the Intune Connector for Active Directory creates on-premises computer objects during hybrid Autopilot join (Phase 4). Defining it now means the target container already exists when the Autopilot deployment profile is configured.

## Design Note

This structure is intentionally simple for a lab but mirrors the *shape* of a production tiered model. In a real environment, the tiering would typically extend further (for example, Tier 0 / Tier 1 / Tier 2 administrative separation for privileged access management), but the principle — separating objects by function to enable scoped sync, scoped policy, and scoped delegation — is the same.

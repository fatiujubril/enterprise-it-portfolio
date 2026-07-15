# Systems Administration & IT Engineering Projects

Enterprise systems engineering built to demonstrate how infrastructure and endpoint decisions directly support secure, identity-driven operations.

Identity runs on infrastructure. These projects emphasize the operational layer beneath access control — device lifecycle, hybrid identity infrastructure, and automated provisioning — with security governance designed in from the start rather than bolted on afterward.

---

## Project 01 — Hybrid Autopilot & Identity-Driven Device Provisioning
**Status: Complete**

Zero-touch Windows device provisioning built end to end on hybrid identity infrastructure. A new device ships sealed to an employee, and on first sign-in it is domain joined, Entra ID registered, enrolled in Intune, and provisioned with department-specific apps, policies, and access — with no IT technician touching the machine.

Built across six phases:

- **On-prem foundation** — AD DS domain controller, tiered OU design, department-attributed user accounts
- **Identity synchronization** — Entra Connect deployment, OU-scoped sync, Password Hash Sync, least-privilege accounts throughout
- **Hybrid Azure AD Join** — dual join state, SCP configuration, and server-registration hardening
- **Windows Autopilot (hybrid profile)** — hardware hash registration, the MSA-based Intune Connector for Active Directory, offline domain join, deployment profile and Enrollment Status Page
- **Identity-driven provisioning** — department dynamic groups driving app assignment, with an attribute-less account as a deliberate negative test
- **Full-flow test & break/fix** — a device provisioned end to end from bare OOBE, validated across every layer

**Validated end to end:** hardware hash → Autopilot profile → offline domain join → Entra registration → Intune enrollment → department app delivered to a Standard user, in under six minutes of provisioning time.

**Four break/fixes diagnosed to root cause**, including a stale AD attribute silently re-seeding deleted device records through the sync engine, and a dormant DHCP misconfiguration introduced in Phase 1 that only surfaced five phases later — because Autopilot is the first workflow that provisions a device booting on DHCP.

**Demonstrates:** AD DS · Entra Connect · Hybrid Azure AD Join · Windows Autopilot · Microsoft Intune · Dynamic Groups · Conditional Access design · PowerShell · Troubleshooting Methodology

📂 [View Project](./project-01-hybrid-autopilot-identity-provisioning/README.md)

---

## Planned Projects

Future work in this section continues the identity lifecycle and secure operations theme, including joiner/mover/leaver automation, administrative tiering, and endpoint baseline hardening.

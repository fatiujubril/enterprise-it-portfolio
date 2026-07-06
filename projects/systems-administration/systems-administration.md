# Systems Administration & IT Engineering Projects

Enterprise systems engineering projects built to demonstrate how infrastructure and endpoint decisions directly support secure, identity-driven operations.

These projects emphasize the operational side of enterprise IT — device lifecycle, hybrid identity infrastructure, and automated provisioning — with security governance built in from the start rather than bolted on afterward.

---

## Project 01 — Hybrid Autopilot & Identity-Driven Device Provisioning
**Status: In Progress**

Zero-touch Windows device provisioning built end-to-end on hybrid identity infrastructure. A new device ships sealed to an employee, and on first sign-in it is domain joined, Entra ID registered, enrolled in Intune, and provisioned with department-specific apps, policies, and access — with no IT technician touching the machine.

The build covers the full hybrid identity chain:

- **On-prem foundation** — AD DS domain controller, OU design, department-attributed user accounts
- **Identity synchronization** — Entra Connect deployment, OU-scoped sync, Password Hash Sync
- **Hybrid Azure AD Join** — dual join state, sync timing dependencies, DC line-of-sight requirements
- **Windows Autopilot (hybrid profile)** — hardware hash registration, Intune Connector for Active Directory, offline domain join
- **Identity-driven provisioning** — dynamic groups by department attribute driving app assignment, configuration profiles, and compliance policies
- **Access governance** — Conditional Access scoped by group and device compliance, PIM for administrative roles
- **Break/fix engineering** — deliberate failure injection with documented diagnosis and resolution

**Demonstrates:** AD DS · Entra Connect · Hybrid Azure AD Join · Windows Autopilot · Microsoft Intune · Dynamic Groups · Conditional Access · PIM · PowerShell · Troubleshooting Methodology

📂 [View Project](./project-01-hybrid-autopilot-identity-provisioning/README.md)

---

## Planned Projects

Future work in this section will continue the identity lifecycle and secure operations theme, including joiner/mover/leaver automation, administrative tiering, and endpoint baseline hardening.

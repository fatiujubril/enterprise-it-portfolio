# Authentication Method Decision — Password Hash Synchronization

## The decision

Entra Connect offers three authentication methods for how synced users sign in to the cloud. **Password Hash Synchronization (PHS)** was selected for this build.

## The options

### Password Hash Synchronization (PHS) — selected

A hash of the user's password hash is synchronized to Entra ID, and the cloud authenticates users directly. The on-premises password (or a reversible form of it) never leaves the premises — only a one-way, re-hashed representation.

**Why chosen:**
- **Simplicity** — no additional on-premises infrastructure beyond Entra Connect itself.
- **Resilience** — because the cloud authenticates independently, users can still sign in to cloud resources even if the on-premises environment is offline. Neither PTA nor Federation offers this.
- **Security feature enablement** — PHS is a prerequisite for Entra ID Identity Protection features such as leaked-credential detection, which compares synced credentials against known-breached credential sets. This aligns directly with the security focus of the wider portfolio.
- **Modern default** — PHS is Microsoft's recommended default and the most widely deployed method.

### Pass-Through Authentication (PTA) — not chosen

Passwords are validated in real time against on-premises AD via a lightweight agent; no password hashes are stored in the cloud.

**Why not:** PTA exists primarily for organizations with a policy or compliance requirement that password material must never reside in the cloud in any form. It introduces an on-premises dependency for *every* sign-in (the agent must be reachable), reducing resilience. No such requirement exists in this environment, so the added dependency is not justified.

### Federation (AD FS) — not chosen

A full on-premises federation infrastructure (AD FS) handles authentication, with Entra ID redirecting sign-ins to the on-premises federation service.

**Why not:** Federation is the most complex and highest-maintenance option, requiring dedicated federation servers and web application proxies. It was historically used for advanced scenarios (smart-card auth, complex claims rules, third-party MFA). Microsoft has spent years moving customers *away* from AD FS toward cloud authentication. For a modern hybrid identity build, it would be an anti-pattern.

## Decision logic

The decision tree an engineer should be able to articulate:

1. **Is there a hard requirement that no password material may exist in the cloud?** If yes → PTA. If no → continue.
2. **Is there a complex federation or claims requirement that only AD FS can satisfy?** If yes → Federation. If no → continue.
3. **Otherwise → PHS** — the simplest, most resilient, most feature-enabling default.

For this build, the answer to (1) and (2) is no, so PHS is the correct choice — and it actively enables the leaked-credential detection capability that complements the security posture the portfolio demonstrates.

---

[← Back to Phase 2 README](./README.md) | [Project overview](../README.md)

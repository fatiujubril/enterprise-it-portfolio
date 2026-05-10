# Conditional Access Baseline

This document captures the **expected secure Conditional Access state** of the tenant prior to any misconfiguration, exception handling, or attack simulation.

The baseline represents the organization’s intended identity security posture, against which all subsequent deviations are measured.

All screenshots in this section are prefixed with `P2-BL-*`.

---

## Baseline Policy Design

At baseline, Conditional Access was configured to enforce multifactor authentication (MFA) consistently across the tenant using a **single, clearly scoped policy**.

The baseline policy was designed with the following principles:

- MFA enforcement for all interactive user sign-ins
- Group-based targeting for scalability and governance
- No user, role, or device-based exclusions
- Policy enabled and actively enforcing (not report-only)

This approach reflects a common and recommended Conditional Access deployment pattern for small-to-medium enterprise environments.

---

## Baseline Enforcement State

Before any misconfiguration occurred:

- Conditional Access evaluated successfully for all users
- MFA challenges were consistently applied during sign-in
- No accounts bypassed authentication requirements
- No emergency or exception-based access paths were present

This confirmed that Conditional Access enforcement was **functioning as intended** prior to the incident scenario.

---

## Baseline Evidence

The following evidence documents the Conditional Access configuration in its secure baseline state.

### Evidence 2.1 – Conditional Access Policy List (Baseline)

This screenshot shows the active Conditional Access policies configured in the tenant at baseline.  
It confirms the presence of a defined MFA enforcement policy prior to any exclusions or modifications.

![Evidence 2.1 – Conditional Access Policy List](../evidence/screenshots/baseline/P2-BL-CA-Policy-List-02.png)

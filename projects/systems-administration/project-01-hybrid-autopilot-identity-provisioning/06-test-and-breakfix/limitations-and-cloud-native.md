# Limitations & When to Choose Cloud-Native Instead

Documenting the boundaries of an architecture matters as much as documenting what works. Hybrid Autopilot succeeds cleanly for **on-network** provisioning — but it has one hard limitation that shapes real deployment decisions, and understanding it is the difference between knowing *how* hybrid works and knowing *when not to use it*.

---

## The remote-worker problem

**The question:** what happens if the device being onboarded through Autopilot has no line of sight to a domain controller at provisioning time — a remote worker unboxing a laptop at home, for example?

**The answer:** the user sign-in fails, even though the domain join itself succeeds. This is not a misconfiguration. It is an architectural consequence of how hybrid join works, and it is the well-known weak point of hybrid Autopilot.

### Why the join succeeds but the logon fails

The two halves of hybrid provisioning have very different network requirements:

- **The offline domain join does NOT need the device to reach a DC.** The Intune Connector (on SYNC01) creates the computer object and generates the ODJ blob *server-side*; the blob travels to the device through Intune. The device applies it and becomes domain-joined over nothing but its internet connection. A remote device completes this step fine.
- **The first interactive sign-in DOES need the device to reach a DC.** Signing in as a domain user (`abello@fjlab.ca`) is a real **Kerberos authentication against a `corp.lab` domain controller**. A device on a home network has no route to internal DCs, cannot obtain a Kerberos ticket, and the user is stranded — on a machine that is technically domain-joined but cannot authenticate its domain user.

This is the same failure signature this project encountered in its [break/fix case study](./breakfix-case-study.md) ("your domain isn't available") — but there it had a fixable DNS cause on an *on-network* device. For a genuinely remote device, the identical symptom has an architectural cause and no simple fix.

---

## The three real responses

**1. Pre-logon VPN (device tunnel).** Ship a VPN client in the deployment profile and bring up a *device-tunnel* VPN during the ESP/device phase, so a DC becomes reachable before the user authenticates. This is the scenario the deployment profile's **"Skip AD connectivity check"** setting exists for — set to **Yes** for remote hybrid deployments, acknowledging the DC will not be reachable at check time and relying on the VPN to bridge it. (This project set it to **No**, correct for an on-network lab.) It works, but it is fragile: the VPN must establish at exactly the right OOBE moment, before a user tunnel would normally exist.

**2. First logon on-network, then cached credentials.** Once a domain user has successfully signed in on-network once, credentials cache and later remote logons succeed. But this requires the first login to happen on-network — which defeats zero-touch onboarding *to* a remote worker.

**3. Use cloud-native (Entra-joined) Autopilot instead — the answer the industry converged on.** A cloud-native device authenticates entirely against Entra ID, needs no DC line of sight *ever*, and provisions over any internet connection. On-prem resource access from that device is then handled by **Cloud Kerberos Trust (Entra Kerberos)**, which lets an Entra-joined device obtain Kerberos tickets for on-prem resources without the device itself being domain-joined.

---

## The decision framework

This limitation is a significant part of *why the industry is moving from hybrid to cloud-native* for new deployments. Hybrid exists to serve environments with genuine on-premises dependencies — but it carries this remote-onboarding cost, and cloud-native eliminates it while still supporting on-prem resource access.

- **Choose hybrid** when there is a real dependency on domain-joined devices (legacy line-of-business apps requiring domain membership, on-prem-only authentication, GPO-based management mid-migration) *and* provisioning happens on-network.
- **Choose cloud-native** for new deployments, remote-first workforces, or any scenario where devices provision off-network — and use Cloud Kerberos Trust to retain on-prem resource access.

This project deliberately built the *harder* hybrid path, to demonstrate the full identity chain and the connector-based offline domain join end to end. In a greenfield or remote-first deployment, cloud-native Entra join would be the recommended choice — and being able to say *why*, precisely, is the point of documenting the limitation rather than omitting it.

---

[← Back to Phase 6 README](./README.md) | [Break/fix case study](./breakfix-case-study.md) | [Project overview](../README.md)

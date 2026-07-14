# Break/Fix Case Study — "Your Domain Isn't Available" at First Hybrid Sign-In

This is the signature failure of the whole project: the single most common hybrid Autopilot break, encountered live during the full-flow test, with a root cause that had lain dormant since Phase 1. It is documented in depth because the *diagnostic path* — not the fix — is the point.

---

## Symptom

During the full-flow Autopilot test, provisioning progressed correctly through the Enrollment Status Page — **Device preparation completed, Device setup completed** — the device rebooted, and then at the first interactive sign-in it failed with:

> **Other user**
> We can't sign you in with this credential because your domain isn't available. Make sure your device is connected to your organization's network and try again.

📸 [Domain not available error](./screenshots/breakfix-01-domain-not-available.png)

Crucially, this appeared **after** Device setup completed — meaning the offline domain join itself had *succeeded* (the `LAB-6KMEnXs2vpf` object was created and applied). The failure was specifically at the point where the device first needed to authenticate a domain user against a live domain controller.

---

## Why this failure surfaces exactly here (the architecture)

Hybrid Autopilot has an asymmetry that makes this the classic failure point:

- **The offline domain join does NOT require the device to reach a DC.** The ODJ connector (on SYNC01) creates the computer object and generates the join blob server-side; the device just applies it. This is why Device setup completed without any DC line-of-sight from the device.
- **The first interactive domain sign-in DOES require the device to reach a DC.** Signing in as `abello@fjlab.ca` — a domain account — is a real Kerberos authentication against a `corp.lab` domain controller. For that, the device must resolve `corp.lab` via DNS and reach DC01.

So a device can sail through the entire connector-driven join and then fail at the very first logon if it cannot locate the domain. "Domain isn't available" is precisely that: the device is domain-joined but cannot find the domain to authenticate against.

---

## Diagnosis — two layers, one red herring

### First finding (partial, misleading): stale DHCP scope option

Inspection of DC01's DHCP showed **Scope Option 006 (DNS Servers) = `10.10.10.10`** — the DC's *original* IP from before it was renumbered to `10.0.0.5` in Phase 1. The scope option had never been updated after the renumber.

📸 [DHCP scope option pointing at the old DC IP](./screenshots/breakfix-02-dhcp-scope-wrong-dns.png)

This looked like the answer — a DHCP client handed a dead DNS server would indeed fail to resolve `corp.lab`. The scope option was corrected to `10.0.0.5`... and the device **still failed**. The stale scope option was real, but it was not the operative cause, because the device was not getting its DHCP from DC01 at all.

### Root cause: an unmanaged DHCP layer on the external switch

`ipconfig /all` on the provisioned device revealed the actual topology problem:

📸 [ipconfig showing DHCP and DNS from the home router](./screenshots/breakfix-03-ipconfig-rogers-dhcp-dns.png)

- **DHCP Server: `10.0.0.1`** — the home gateway (Rogers router), *not* DC01 (`10.0.0.5`).
- **DNS Servers: Rogers public DNS** (IPv4 `64.71.255.x` and IPv6 `2607:f798:...`) — servers with no knowledge of `corp.lab`.
- **Connection-specific DNS suffix: `phub.net.cable.rogers.com`** — confirming the lease came from the ISP router.

The `vSwitch-EXT-HOME` external switch bridges the lab VMs onto the physical home LAN, where the **Rogers router answers DHCP** — and wins the race against DC01's DHCP. Every DHCP-booting client gets the router's IP, gateway, and public DNS. Correcting DC01's scope option changed nothing because nothing was asking DC01 for DHCP.

A secondary complication: even after a static IPv4 DNS entry for `10.0.0.5` was added, resolution still failed because **IPv6 DNS took precedence** — `nslookup` was still querying a Rogers IPv6 server. IPv6 had to be overridden as well.

### Why this stayed dormant until Phase 6

Every prior phase worked because DC01, SYNC01, and the original WIN11 client were configured with **static IPs and static DNS pointing at 10.0.0.5**. Nothing in Phases 1–5 depended on *DHCP-delivered* DNS to reach the domain controller. **Autopilot is the first workflow that provisions a device which boots on DHCP** — so it is the first thing that ever exercised, and therefore exposed, the unmanaged DHCP layer. A misconfiguration can hide indefinitely until a new dependency finally relies on the thing it broke.

---

## Resolution

### Immediate (unblock the test)

Static IPv4 configuration was forced on the device (via `cmd` reached through the OOBE utilman/accessibility path), overriding both the router's DHCP and IPv6 DNS precedence:

```cmd
netsh interface ip set address name="Ethernet" static 10.0.0.20 255.255.255.0 10.0.0.1
netsh interface ip set dns name="Ethernet" static 10.0.0.5 primary
```

Verified resolution by querying DC01 directly over IPv4 (`nslookup corp.lab 10.0.0.5`), then restarted the sign-in. Authentication succeeded and provisioning continued to the desktop.

📸 [Domain sign-in succeeds — profile building](./screenshots/breakfix-04-domain-signin-success.png)

### Proper remediation (backlog)

The static-IP fix is a workaround, not a cure — and importantly, **it does not scale to real zero-touch provisioning**, because a sealed device cannot be hand-configured via utilman. The disease is two DHCP servers on one L2 segment with the wrong one winning. The correct lab fix is to isolate the VMs from the home LAN:

- Move the lab to an **internal or private Hyper-V switch** so DC01 is the sole DHCP/DNS authority, with internet egress routed deliberately (through DC01 or a second NIC) rather than via a bridged ISP router.
- Correct the stale Scope Option 006 regardless (done) so DC01's own DHCP is clean when it becomes authoritative.

---

## Takeaways

1. **Know where a hybrid Autopilot failure sits in the flow.** Failure *after* Device setup but *at first sign-in* points at domain reachability, not the join — because the connector-driven ODJ needs no device-to-DC path, but the interactive logon does. The location of the failure in the sequence is itself a diagnostic.
2. **The first plausible cause is not always the operative one.** The stale scope option was a genuine misconfiguration and a tempting answer; fixing it and re-testing (rather than assuming) is what revealed the device wasn't using DC01's DHCP at all. Verify the fix actually changed the outcome before closing.
3. **Dormant misconfigurations surface when a new dependency first relies on them.** The lab's DNS had always quietly depended on static configuration; Autopilot, uniquely, provisions a DHCP-booting device, so it was the first workflow to expose the unmanaged DHCP layer — years of "working" masked a latent gap.
4. **A workaround that doesn't scale is not a fix.** Static-IP-via-utilman rescued the test but is impossible on a sealed device — so the real remediation is network redesign, and saying so plainly is part of an honest post-mortem.

---

[← Back to Phase 6 README](./README.md) | [Full-flow timing log](./full-flow-timing-log.md) | [Project overview](../README.md)

# Zero Trust Design
## FatiuLab Security — Zero Trust Architecture & Implementation

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Framework:** Zero Trust Architecture (NIST SP 800-207)  
**Last Updated:** May 2026

---

## 1. Zero Trust Principles

Zero Trust is built on three core principles that together eliminate implicit trust from every layer of the security architecture:

| Principle | Definition | Implementation at FatiuLab Security |
|---|---|---|
| Verify Explicitly | Always authenticate and authorize using all available signals | MFA on every authentication via CA01/CA02/CA04 |
| Use Least Privilege | Limit access to only what is needed, when it is needed | PIM JIT access — no standing privilege for any admin role |
| Assume Breach | Design as if the attacker is already inside | Sentinel detection rules, network isolation, audit logging |

> "Zero Trust is not a product you buy. It is an architectural philosophy you implement layer by layer across identity, devices, network, applications, and data."

---

## 2. Zero Trust Pillars — Coverage Map

NIST SP 800-207 defines Zero Trust across six pillars. The following maps FatiuLab Security's implementation to each pillar:

| Pillar | Coverage | Implementation |
|---|---|---|
| Identity | ✅ Full | Entra ID P2, MFA, PIM, Access Reviews, CA policies |
| Devices | ⚠️ Partial | MFA required — device compliance not yet enforced |
| Network | ✅ Partial | Storage account network isolation, legacy auth blocked |
| Applications | ✅ Partial | CA policies scope specific apps (Azure Management) |
| Data | ✅ Partial | Storage encryption, shared key access disabled |
| Infrastructure | ✅ Partial | Defender for Cloud CSPM, CIS benchmark |

**Focus area:** This program delivers full Zero Trust coverage for the Identity pillar — the highest-risk and highest-value pillar for a cloud-native FinTech. Device and network pillars are partially covered with clear improvement roadmap items.

---

## 3. Zero Trust Identity Architecture

Identity is the new perimeter in a cloud environment. The traditional network perimeter — firewall, VPN, corporate network — no longer provides meaningful protection when users access resources from anywhere on any device. Identity is the only consistent control plane.

```
OLD MODEL (perimeter-based):
Corporate Network = Trusted
Outside Network   = Untrusted

Flaw: Once inside the network, everything is trusted.
      Lateral movement is trivial.
      Stolen credentials + VPN = full access.

ZERO TRUST MODEL (identity-based):
Every request = Untrusted until verified
Every authentication = Challenge with MFA
Every privilege = Time-limited and justified
Every action = Logged and monitored
```

---

## 4. Verify Explicitly — Implementation

### 4.1 Conditional Access Policy Set

Every authentication request is evaluated against the CA policy set before access is granted. No user — including admins — authenticates without MFA.

| Policy | Scope | Control | Rationale |
|---|---|---|---|
| CA01 | All users | Require MFA | Baseline — no exceptions |
| CA02 | Admin roles | Require MFA | Elevated enforcement for privileged accounts |
| CA03 | All users | Block legacy auth | Eliminate MFA bypass via legacy protocols |
| CA04 | All users | Require MFA for Azure Management | Separate challenge for cloud infrastructure access |
| CA05 | Standard users | Require MFA (report-only) | Staged rollout — pending MFA registration |

**Signal evaluation:** Every authentication is evaluated against:
- User identity and role
- Application being accessed
- Authentication method used (modern vs legacy)
- Client app type

**Named location:** `LOC-FatiuLab-HQ` (192.168.1.0/24) defined as trusted network. Future enhancement: require device compliance for access outside trusted network.

Evidence: [P3-CA-00-Policy-List.png](../Evidence/Phase-01-Identity/CA-Policies/P3-CA-00-Policy-List.png)

### 4.2 Break-Glass Exception

The break-glass account (`breakglass@FatiuLabSecurity.onmicrosoft.com`) is excluded from all CA enforcement policies via `SEC-CA-Exclusions`. This is a deliberate, documented exception — not a gap. The account exists to recover the tenant if all other authentication mechanisms fail.

Compensating controls:
- Credentials stored offline
- Monthly access review of SEC-CA-Exclusions (AR-02)
- NRT detection rule (DET-07) — any sign-in generates immediate alert

---

## 5. Use Least Privilege — Implementation

### 5.1 Privileged Identity Management

Standing privilege is eliminated for all privileged roles. No admin account has always-on elevated access except the Global Admin safety anchor and break-glass account.

```
WITHOUT LEAST PRIVILEGE:
Admin has Global Admin 24/7
Attacker compromises account → unlimited access forever

WITH LEAST PRIVILEGE (PIM):
Admin is eligible for Global Admin
Attacker compromises account → no elevated access

To get elevated access attacker must also:
- Submit activation request
- Provide justification
- Complete MFA challenge
- Get approval from second person
- All within a 2-hour window
```

| Role | Activation Duration | Approval | Eligible Expiry |
|---|---|---|---|
| Global Administrator | 2 hours | Required | 3 months |
| Security Administrator | 4 hours | Self-approve | 6 months |

Evidence: [P3-PIM-03-Eligible-Assignments.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03-Eligible-Assignments.png)

### 5.2 Access Reviews

Least privilege decays over time without active governance. Access reviews enforce periodic revalidation — ensuring access is still needed, not just assumed.

| Review | Frequency | Groups | Auto-Remove |
|---|---|---|---|
| AR-01 | Quarterly | SEC-Global-Admins, SEC-Admins, SEC-CA-Exclusions | Yes |
| AR-02 | Monthly | SEC-CA-Exclusions | Yes |

The monthly review of SEC-CA-Exclusions is the most critical governance control — any unauthorized account in this group bypasses all CA policies.

### 5.3 Azure RBAC

Storage account and Azure resources are protected by Entra ID RBAC. Shared key access is disabled — all access must be authenticated via Entra ID identity, not anonymous shared keys.

---

## 6. Assume Breach — Implementation

### 6.1 Detection Engineering

8 custom Sentinel analytics rules assume attackers are already inside and monitor for the behavioral indicators of identity-based attacks:

| What We Assume | Detection Rule |
|---|---|
| Attacker using stolen credentials outside hours | DET-01 PIM activation outside business hours |
| Attacker escalating via PIM bypass | DET-02 Global Admin assigned outside PIM |
| Attacker weakening MFA controls | DET-03 MFA disabled |
| Attacker disabling CA policies | DET-04 CA policy modified |
| Attacker using credentials from foreign location | DET-05 Impossible travel |
| Attacker gaining Azure control plane access | DET-06 Azure role assignment |
| Attacker using break-glass credentials | DET-07 Break-glass sign-in (NRT) |
| Attacker exploiting cloud misconfigurations | DET-08 Defender high severity |

### 6.2 Network Isolation

Even with Owner access on the Azure subscription, an attacker cannot access the storage account — public network access is disabled entirely.

```
Attacker compromises account with Owner access
        ↓
Attacker attempts to access fatiulabstorage01
        ↓
Public network access: Disabled
        ↓
Access denied — network isolation holds
```

This defense-in-depth approach means a compromised identity does not automatically equal data access.

### 6.3 Audit Trail

Every privileged action generates an audit record. The audit trail is immutable and centralized in Sentinel — an attacker who compromises an admin account cannot delete the evidence of what they did.

| Action | Audit Source | Sentinel Table |
|---|---|---|
| PIM activation | Entra ID audit | AuditLogs |
| CA policy change | Entra ID audit | AuditLogs |
| Sign-in event | Entra ID sign-in | SigninLogs |
| Azure role assignment | Azure Activity | AzureActivity |
| Resource deployment | Azure Activity | AzureActivity |

---

## 7. Zero Trust Maturity Assessment

Using the CISA Zero Trust Maturity Model (Traditional → Initial → Advanced → Optimal):

| Pillar | Current Maturity | Target |
|---|---|---|
| Identity | **Advanced** — MFA, PIM, access reviews, CA policies | Optimal — add phishing-resistant MFA |
| Devices | **Initial** — MFA required, no device compliance | Advanced — add Intune compliance policy |
| Network | **Initial** — legacy auth blocked, storage isolated | Advanced — add private endpoints, network segmentation |
| Applications | **Initial** — Azure Management scoped | Advanced — per-app CA policies |
| Data | **Initial** — encryption at rest and transit | Advanced — add sensitivity labels |
| Infrastructure | **Advanced** — CSPM, CIS benchmark, SIEM | Optimal — add infrastructure as code |

**Identity pillar at Advanced maturity** is the primary achievement of this program. This is the correct focus for a cloud-native FinTech — identity is the highest-value pillar because it controls access to everything else.

---

## 8. Zero Trust Improvement Roadmap

| Improvement | Pillar | Priority | Dependency |
|---|---|---|---|
| Phishing-resistant MFA (FIDO2) | Identity | Critical | Hardware keys or compatible devices |
| Device compliance CA policy | Devices | High | Intune MDM deployment |
| Private endpoint for storage | Network | Medium | Azure VNet deployment |
| CA05 enforcement (On) | Identity | High | MFA registration campaign |
| Sensitivity labels | Data | Medium | Microsoft Purview |
| Privileged Access Workstation policy | Devices | Medium | Intune + compliant devices |

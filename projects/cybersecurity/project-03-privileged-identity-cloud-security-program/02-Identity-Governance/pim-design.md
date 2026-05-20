# Privileged Identity Management Design
## FatiuLab Security — Just-In-Time Privileged Access Program

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Regulatory Context:** PCI-DSS, PIPEDA  
**Tenant:** FatiuLabSecurity.onmicrosoft.com  
**Policy Author:** Global Administrator  
**Last Reviewed:** May 2026  
**Status:** Active — Phase 1 Complete

---

## 1. Overview

Privileged Identity Management (PIM) is the technical implementation of the Zero Trust principle **Use Least Privilege Access** — specifically the Just-In-Time (JIT) and Just-Enough-Access (JEA) components. In a FinTech environment processing payment data, standing privileged access represents one of the highest-risk attack surfaces: a compromised admin account with permanent Global Administrator access gives an attacker unrestricted access to the entire tenant, all payment infrastructure, and all customer data — with no time limit.

This document defines the PIM architecture implemented at FatiuLab Security, covering role settings, eligible assignments, activation workflows, and the audit trail that proves the program is operating as designed.

**The core problem PIM solves:**

```
WITHOUT PIM (standing privilege):        WITH PIM (JIT access):
User has Global Admin permanently   →    User has NO privileges by default
No audit trail                           ↓
No time limit                            User requests activation
No approval required                     ↓
Attacker gets forever access if          Justification + MFA + Approval
account is compromised                   ↓
                                         2-hour time-limited access granted
                                         ↓
                                         Full audit trail recorded
                                         ↓
                                         Access expires automatically
```

---

## 2. Design Principles

| Principle | Implementation |
|---|---|
| No standing privilege | All privileged roles eligible only — no permanent active assignments (except break-glass) |
| Just-In-Time access | Role activations are time-limited — 2 hours for Global Admin, 4 hours for Security Admin |
| Justification required | Every activation requires a written business justification — full audit trail |
| Approval for highest-risk roles | Global Administrator requires explicit approval before activation |
| MFA on activation | Identity verified at activation time — not just at login |
| Time-bound eligibility | Eligible assignments expire — forced review every 3-6 months |
| Defense in depth | Direct active assignments also require MFA and justification as a safety net |

---

## 3. Roles Under PIM Management

| Role | Risk Level | Activation Duration | Approval Required | MFA Required | Eligible Expiry |
|---|---|---|---|---|---|
| Global Administrator | Critical | 2 hours | Yes | Yes | 3 months |
| Security Administrator | High | 4 hours | No (self-approve) | Yes | 6 months |

**Why different settings for each role:**

**Global Administrator** is the highest-privilege role in the entire tenant — full control over identity, security configuration, billing, and all Microsoft 365 and Azure resources. A compromised Global Admin account in a payment processing environment is a catastrophic breach. The 2-hour limit and mandatory approval reflect the severity of this risk.

**Security Administrator** has broad security tooling access — Defender XDR, Sentinel, Conditional Access, PIM itself — but cannot modify billing, create new admins, or access all tenant resources. The 4-hour limit reflects the reality that security investigations take longer than 2 hours, and self-approval is acceptable because the Security Admin role is lower risk than Global Admin.

---

## 4. Role Settings

### 4.1 Global Administrator Role Settings

**Activation tab:**

| Setting | Value | Rationale |
|---|---|---|
| Activation maximum duration | 2 hours | Minimizes exposure window for highest-risk role |
| Require MFA on activation | Yes | Identity verification at time of privilege use |
| Require justification | Yes | Audit trail for every use — PCI-DSS requirement |
| Require approval | Yes | Second pair of eyes on highest-risk role activations |
| Approvers | Global Admin account | Designated approver for all GA activations |

**Assignment tab:**

| Setting | Value | Rationale |
|---|---|---|
| Allow permanent eligible | No | Eligibility must be reviewed and renewed |
| Expire eligible after | 3 months | Quarterly governance review cadence |
| Allow permanent active | No | No standing privilege — ever |
| Expire active after | 15 days | Safety net on direct assignments |
| Require MFA on active assignment | Yes | Extra layer on direct assignment backdoor |
| Require justification on active assignment | Yes | Audit trail even on direct assignments |

Evidence: [P3-PIM-01-GlobalAdmin-Role-Settings.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-01-GlobalAdmin-Role-Settings.png)

---

### 4.2 Security Administrator Role Settings

**Activation tab:**

| Setting | Value | Rationale |
|---|---|---|
| Activation maximum duration | 4 hours | Reflects longer security investigation windows |
| Require MFA on activation | Yes | Identity verification at time of privilege use |
| Require justification | Yes | Audit trail for every use |
| Require approval | No | Security team self-approves — lower risk than GA |

**Assignment tab:**

| Setting | Value | Rationale |
|---|---|---|
| Allow permanent eligible | No | Eligibility must be reviewed and renewed |
| Expire eligible after | 6 months | Semi-annual governance review cadence |
| Allow permanent active | No | No standing privilege |
| Expire active after | 15 days | Safety net on direct assignments |
| Require MFA on active assignment | Yes | Defense in depth on direct assignments |
| Require justification on active assignment | Yes | Audit trail on direct assignments |

Evidence: [P3-PIM-02-SecurityAdmin-Role-Settings.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-02-SecurityAdmin-Role-Settings.png)

---

## 5. Eligible Assignments

Eligible assignments define who is permitted to request activation of a privileged role. Being eligible does not grant any privileges — it only grants the ability to request access through the JIT workflow.

| User | Role | Assignment Type | Start Date | Expiry | Justification |
|---|---|---|---|---|---|
| securityadmin@ | Global Administrator | Eligible | May 2026 | Aug 2026 (3 months) | Break-glass escalation path for Security Admin |
| securityadmin@ | Security Administrator | Eligible | May 2026 | Nov 2026 (6 months) | Primary role assignment via PIM |

**Design decisions:**

- Security Admin is eligible for Global Administrator as an escalation path — if a critical incident requires GA-level access and the primary Global Admin account is unavailable, Security Admin can escalate through PIM with approval
- Security Admin is eligible for their own primary role (Security Administrator) — this is the standard operational JIT pattern
- Global Admin account holds a permanent active Global Administrator assignment — this is the intentional safety anchor for the tenant

Evidence:
- [P3-PIM-03-Eligible-Assignments.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03-Eligible-Assignments.png) — both eligible assignments in list
- [P3-PIM-03b-Eligible-Assignment-Settings.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03b-Eligible-Assignment-Settings.png) — eligible assignment creation settings
- [P3-PIM-03d-Active-Assignments.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03d-Active-Assignments.png) — active assignments — Global Admin permanent only

---

## 6. Active Assignments

| User | Role | Assignment Type | Rationale |
|---|---|---|---|
| globaladmin@ | Global Administrator | Permanent active | Tenant safety anchor — intentional standing privilege |

**Why one permanent active assignment is correct:**

Zero Trust does not mean zero permanent access. It means zero unnecessary permanent access. The Global Admin account requires permanent active Global Administrator for two reasons:

1. **PIM dependency** — PIM itself requires at least one permanent Global Admin to process activation approvals and manage the PIM configuration. If all Global Admin access was JIT, a PIM outage could make it impossible to approve activation requests, locking out all privileged access.

2. **Break-glass equivalent** — The permanent Global Admin is the last-resort recovery account for the tenant, equivalent in function to the break-glass account but for role access rather than CA policy bypass.

Security Admin was removed from permanent active assignment and converted to eligible-only — this is the correct JIT posture for operational privileged accounts.

Evidence: [P3-PIM-03c-Active-Assignment-Enforcement.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03c-Active-Assignment-Enforcement.png)

---

## 7. JIT Activation Workflow

This section documents the end-to-end JIT activation workflow as tested in the lab environment.

### 7.1 Workflow Steps

```
Step 1 — User navigates to PIM > My roles
         Security Admin sees eligible roles listed
         ↓
Step 2 — User clicks Activate next to Security Administrator
         Activation request form opens
         ↓
Step 3 — User enters:
         Duration: 4 hours
         Justification: Business reason for access
         ↓
Step 4 — MFA challenge issued
         User completes MFA verification
         ↓
Step 5 — Role activated
         Security Administrator now shows as Active
         Expiry timestamp set: activation time + 4 hours
         ↓
Step 6 — Audit event recorded
         PIM audit log + Entra ID audit log both capture the event
         Justification text visible in audit record
         ↓
Step 7 — Expiry
         After 4 hours role activation expires automatically
         User returns to eligible-only state
         New activation required for further privileged access
```

### 7.2 Activation Test Evidence

**Before activation — eligible roles:**

Security Admin signed in and navigated to PIM > My roles. Both eligible roles visible with Activate button available — no privileges active.

Evidence: [P3-PIM-04-JIT-My-Roles-Eligible.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-04-JIT-My-Roles-Eligible.png)

**Activation request:**

Activation request submitted with 4-hour duration and justification: *"Investigating suspicious sign-in activity - routine security review for lab documentation."*

Evidence: [P3-PIM-05-JIT-Activation-Request.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-05-JIT-Activation-Request.png)

**Activation confirmed:**

Security Administrator role shows as Active. Expiry timestamp confirms 4-hour window from activation time. Deactivate button visible — role is live.

Evidence: [P3-PIM-06-JIT-Activation-Confirmed.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-06-JIT-Activation-Confirmed.png)

---

## 8. Audit Trail

PIM generates two separate audit records for every privileged action — the PIM activity log and the Entra ID audit log. Both are captured as evidence.

### 8.1 PIM Activity Log

Records all administrator actions — role settings changes, assignment creation and removal. Shows the full governance history of the PIM program from initial configuration through ongoing management.

Key events recorded:
- Global Administrator role settings updated
- Security Administrator role settings updated
- Security Admin eligible assignment created (Global Administrator)
- Security Admin eligible assignment created (Security Administrator)
- Security Admin removed from permanent active assignment

Evidence: [P3-PIM-07-Audit-Log.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-07-Audit-Log.png)

### 8.2 Entra ID Audit Log

Records all identity events including user-initiated JIT activations. The justification text entered by the user at activation time is captured in the audit record — this is the evidentiary trail that proves access was justified and documented at the time of use.

Key events recorded:
- PIM — Add member to role completed (Security Administrator activation)
- Status: Success
- Justification text visible in audit record

Evidence: [P3-PIM-07b-Activation-Audit-Log.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-07b-Activation-Audit-Log.png)

---

## 9. Privileged Access Operating Model

This section defines the operational procedures for privileged access at FatiuLab Security.

### 9.1 Normal Operations
- All privileged work performed under JIT-activated roles
- Activations limited to task duration — deactivate early when work is complete
- Every activation requires a specific, meaningful justification — not generic text
- Activation window selection: use minimum duration needed, not maximum available

### 9.2 Approval Process (Global Administrator)
1. Security Admin submits GA activation request with justification
2. Approver (Global Admin) receives notification
3. Approver reviews justification — approves or denies
4. If approved: Security Admin receives MFA challenge
5. On MFA completion: GA access granted for up to 2 hours
6. All steps recorded in audit log

### 9.3 Break-Glass Escalation
If PIM is unavailable or all eligible accounts are inaccessible:
1. Retrieve break-glass credentials from secure offline storage
2. Sign in as breakglass@FatiuLabSecurity.onmicrosoft.com
3. Break-glass account has permanent active Global Administrator — bypasses PIM entirely
4. Document the use immediately — break-glass use is a security event
5. Review and investigate root cause before returning to normal operations
6. Break-glass use triggers AR-02 review of SEC-CA-Exclusions group

### 9.4 Eligibility Review Cadence
| Role | Review Frequency | Reviewer | Action if No Response |
|---|---|---|---|
| Global Administrator | Quarterly (3 months) | Global Admin | Remove eligibility |
| Security Administrator | Semi-annual (6 months) | Global Admin | Remove eligibility |

Reviews are automated via Entra ID Access Reviews — see [access-reviews.md](access-reviews.md) for full configuration details.

---

## 10. Gaps and Planned Improvements

| Gap | Risk | Planned Remediation | Priority |
|---|---|---|---|
| No PIM alerts configured | Unusual activation patterns not alerted | Configure PIM alerts for suspicious patterns | High |
| Privileged Role Administrator not under PIM | Can modify PIM settings without JIT | Add Privileged Role Admin to PIM scope | High |
| No Sentinel integration yet | PIM audit events not feeding SIEM | Connect Entra ID logs to Sentinel — Week 3 | Medium |
| Single approver for GA activations | Approver unavailability blocks all GA access | Add secondary approver | Medium |
| No time-of-day restriction on activations | JIT activations possible outside business hours | Detection rule: PIM activation outside business hours (Sentinel Week 3) | Medium |

---

## 11. Compliance Alignment

| Control | PCI-DSS | PIPEDA | NIST CSF | Zero Trust |
|---|---|---|---|---|
| JIT access — no standing privilege | Req 7.2, 8.2 | Principle 4 | PR.AC-4 | Least privilege |
| MFA on activation | Req 8.4 | Principle 7 | PR.AC-7 | Verify explicitly |
| Justification required | Req 10.2 | Principle 4 | DE.CM-3 | Assume breach |
| Approval for GA activations | Req 7.2 | Principle 4 | PR.AC-4 | Least privilege |
| Time-limited access | Req 7.2 | Principle 4 | PR.AC-4 | Least privilege |
| Eligible assignment expiry | Req 8.1 | Principle 4 | PR.AC-1 | Least privilege |
| Full audit trail | Req 10.2, 10.3 | Principle 4 | DE.CM-3 | Assume breach |

---

## 12. Evidence Index

| File | Description |
|---|---|
| [P3-PIM-00-Quick-Start.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-00-Quick-Start.png) | PIM enabled and ready |
| [P3-PIM-01-GlobalAdmin-Role-Settings.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-01-GlobalAdmin-Role-Settings.png) | Global Admin role settings — 2hr, MFA, approval required |
| [P3-PIM-02-SecurityAdmin-Role-Settings.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-02-SecurityAdmin-Role-Settings.png) | Security Admin role settings — 4hr, MFA, self-approve |
| [P3-PIM-03-Eligible-Assignments.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03-Eligible-Assignments.png) | Both eligible assignments with expiry dates |
| [P3-PIM-03b-Eligible-Assignment-Settings.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03b-Eligible-Assignment-Settings.png) | Eligible assignment creation — 3 month expiry enforced |
| [P3-PIM-03c-Active-Assignment-Enforcement.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03c-Active-Assignment-Enforcement.png) | Active assignment enforcement — MFA + justification required |
| [P3-PIM-03d-Active-Assignments.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-03d-Active-Assignments.png) | Active assignments — Global Admin permanent only |
| [P3-PIM-04-JIT-My-Roles-Eligible.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-04-JIT-My-Roles-Eligible.png) | Security Admin eligible roles before activation |
| [P3-PIM-05-JIT-Activation-Request.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-05-JIT-Activation-Request.png) | JIT activation request with justification |
| [P3-PIM-06-JIT-Activation-Confirmed.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-06-JIT-Activation-Confirmed.png) | Role activated — 4 hour window confirmed |
| [P3-PIM-07-Audit-Log.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-07-Audit-Log.png) | PIM admin audit history — full governance trail |
| [P3-PIM-07b-Activation-Audit-Log.png](../Evidence/Phase-01-Identity/PIM/P3-PIM-07b-Activation-Audit-Log.png) | Entra ID audit log — JIT activation with justification text |

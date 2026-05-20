# Access Reviews Design
## FatiuLab Security — Identity Governance Review Program

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Regulatory Context:** PCI-DSS, PIPEDA  
**Tenant:** FatiuLabSecurity.onmicrosoft.com  
**Policy Author:** Global Administrator  
**Last Reviewed:** May 2026  
**Status:** Active — Phase 1 Complete

---

## 1. Overview

Access Reviews are the governance layer that answers the single most important question in identity security:

> **Does this person still need this access?**

PIM controls how privileged access is used (JIT, time-limited, audited). Conditional Access controls how users authenticate. Access Reviews control whether users should have access at all — catching privilege accumulation, dormant accounts, and role creep before they become security incidents.

In a FinTech environment processing payment data, unchecked access accumulation is a direct PCI-DSS compliance risk. PCI-DSS Requirement 7 mandates that access to system components is restricted to only those individuals whose job requires it — Access Reviews are the operational mechanism that enforces this requirement on an ongoing basis.

**The core problem Access Reviews solve:**

```
WITHOUT Access Reviews:              WITH Access Reviews:
User joins → gets access        →    User joins → gets access
User changes role                    ↓
Old access never removed             Quarterly/monthly review triggered
User leaves                          ↓
Account disabled but group           Reviewer confirms: still needed?
membership stays                     ↓
                                     Yes → access retained
Privilege accumulates                No → access removed automatically
over months and years                ↓
                                     Audit trail of every decision
```

---

## 2. Design Principles

| Principle | Implementation |
|---|---|
| Periodic revalidation | All privileged group memberships reviewed on a defined schedule |
| Auto-remediation | No response from reviewer = access removed automatically |
| Highest-risk group reviewed most frequently | SEC-CA-Exclusions reviewed monthly — bypasses all CA policies |
| Justification required | Reviewers must provide justification for access retention decisions |
| Decision helpers enabled | Inactive accounts (30+ days no sign-in) flagged for removal |
| Full audit trail | Every review decision recorded — who reviewed, what decision, when |

---

## 3. Groups Under Review

| Group | Risk Level | Why Reviewed | Review Frequency |
|---|---|---|---|
| SEC-Global-Admins | Critical | Members have or can access Global Admin privileges | Quarterly |
| SEC-Admins | High | Members have Security Administrator access | Quarterly |
| SEC-CA-Exclusions | Critical | Members bypass ALL Conditional Access policies | Monthly |

**Why SEC-CA-Exclusions is reviewed monthly:**

SEC-CA-Exclusions is the highest-risk group in the entire tenant. Any account in this group bypasses every CA policy — MFA requirements, legacy auth blocking, Azure Management MFA — everything. An attacker who compromises any account in this group has effectively bypassed the entire Zero Trust CA framework.

This group should contain exactly one account at all times: the break-glass account. Monthly review ensures no unauthorized accounts are silently added to this group — a technique used by attackers to maintain persistent access after initial compromise.

---

## 4. Access Review Configuration

### AR-01 — Privileged Groups Review (Quarterly)

**Purpose:** Quarterly revalidation of membership in privileged security groups. Ensures that only current, active employees with a legitimate business need retain membership in groups that grant or enable privileged access.

| Setting | Value |
|---|---|
| Review name | AR-01-Privileged-Groups-Review-Quarterly |
| Resource type | Teams + Groups |
| Groups in scope | SEC-Global-Admins, SEC-Admins, SEC-CA-Exclusions |
| Scope | All users (members) |
| Reviewers | Global Administrator account |
| Review duration | 7 days |
| Recurrence | Quarterly |
| Start date | May 2026 |
| End date | No end date |
| Auto-apply results | Enabled |
| If reviewers don't respond | Remove access |
| Decision helpers | Flag users inactive 30+ days |
| Justification required | Yes |
| Email notifications | Enabled |
| Reminders | Enabled |

**Auto-apply and no-response policy rationale:**

If a reviewer fails to complete the review within 7 days, access is automatically removed. This is intentionally strict — in a PCI-DSS environment, the default posture must be deny. A missed review is treated as "access not justified" rather than "access assumed OK." This prevents reviews from becoming a rubber-stamp exercise where inaction equals approval.

Evidence:
- [P3-AR-01-Privileged-Groups-Review-Settings.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-01-Privileged-Groups-Review-Settings.png) — full review configuration
- [P3-AR-01b-Privileged-Groups-Review-Created.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-01b-Privileged-Groups-Review-Created.png) — review created and active

---

### AR-02 — CA Exclusions Review (Monthly)

**Purpose:** Monthly revalidation of SEC-CA-Exclusions group membership. This is the highest-frequency, highest-priority review in the program — the CA exclusion group bypasses all Conditional Access policies and must be monitored continuously.

| Setting | Value |
|---|---|
| Review name | AR-02-CA-Exclusions-Review-Monthly |
| Resource type | Teams + Groups |
| Groups in scope | SEC-CA-Exclusions |
| Scope | All users (members) |
| Reviewers | Global Administrator account |
| Review duration | 3 days |
| Recurrence | Monthly |
| Start date | May 2026 |
| End date | No end date |
| Auto-apply results | Enabled |
| If reviewers don't respond | Remove access |
| Decision helpers | Flag users inactive 30+ days |
| Justification required | Yes |
| Email notifications | Enabled |
| Reminders | Enabled |

**3-day review duration rationale:**

AR-02 uses a 3-day review window instead of 7 days because the risk profile of SEC-CA-Exclusions is significantly higher than other groups. An unauthorized account in this group bypasses all CA controls — the faster this is caught and remediated, the smaller the exposure window. Monthly frequency with a 3-day window means the maximum time an unauthorized account could remain in this group before forced review is approximately 30 days.

**Expected membership:** Exactly 1 account — breakglass@FatiuLabSecurity.onmicrosoft.com. Any additional members should be treated as a security incident.

Evidence: [P3-AR-02-CA-Exclusions-Review-Settings.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-02-CA-Exclusions-Review-Settings.png)

---

## 5. Review Program Overview

Evidence: [P3-AR-00-Access-Reviews-List.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-00-Access-Reviews-List.png)

| Review | Groups | Frequency | Duration | Auto-Remove | Status |
|---|---|---|---|---|---|
| AR-01-Privileged-Groups-Review-Quarterly | SEC-Global-Admins, SEC-Admins, SEC-CA-Exclusions | Quarterly | 7 days | Yes | Active |
| AR-02-CA-Exclusions-Review-Monthly | SEC-CA-Exclusions | Monthly | 3 days | Yes | Active |

**Note on overlap:** SEC-CA-Exclusions appears in both AR-01 and AR-02. This is intentional — AR-01 provides quarterly deep review alongside other privileged groups, while AR-02 provides monthly dedicated monitoring of the highest-risk group. The more frequently a high-risk group is reviewed, the smaller the window for undetected unauthorized access.

---

## 6. Reviewer Responsibilities

| Role | Responsibility |
|---|---|
| Global Administrator (reviewer) | Complete all assigned reviews within the review window |
| Global Administrator (reviewer) | Provide justification for every access retention decision |
| Global Administrator (reviewer) | Investigate any unexpected memberships before making decisions |
| Global Administrator (reviewer) | Treat any unknown account in SEC-CA-Exclusions as a security incident |

**Review decision guidance:**

| Scenario | Decision | Action |
|---|---|---|
| User is active, role still required | Approve | Retain access |
| User has changed roles, access no longer needed | Deny | Remove access |
| User inactive 30+ days (flagged by decision helper) | Investigate | Verify status before deciding |
| Unknown account in SEC-CA-Exclusions | Deny | Remove immediately, investigate |
| Break-glass account in SEC-CA-Exclusions | Approve | This is expected and correct |

---

## 7. Integration with PIM and Conditional Access

Access Reviews do not operate in isolation — they are the governance layer that sits on top of PIM and Conditional Access:

```
CONDITIONAL ACCESS          PIM                    ACCESS REVIEWS
Controls HOW users     →   Controls WHEN users →   Controls WHETHER users
authenticate               get privileges           should have access at all

CA01: MFA required         JIT activation           Quarterly: does this user
CA02: Admin MFA            2hr/4hr windows          still need this group?
CA03: Block legacy         Approval workflow
CA04: Azure MFA            Audit trail              Monthly: is SEC-CA-Exclusions
CA05: Standard users                                membership still correct?
```

**The complete identity governance loop:**

1. User is assigned to a group (initial provisioning)
2. CA policies enforce authentication requirements based on group membership
3. PIM enforces JIT access for privileged role activations
4. Access Reviews periodically revalidate whether group membership is still appropriate
5. If access is removed → CA policies and PIM eligibility automatically update
6. Audit trail maintained at every step

---

## 8. Gaps and Planned Improvements

| Gap | Risk | Planned Remediation | Priority |
|---|---|---|---|
| No review for SEC-PIM-Eligible group | Unauthorized PIM eligibility not reviewed | Add SEC-PIM-Eligible to AR-01 scope | High |
| No Entra role review | Direct role assignments not reviewed | Add Entra ID role review when available in tenant | High |
| Single reviewer | Reviewer unavailability delays access removal | Add secondary reviewer or fallback | Medium |
| No review for standard user groups | SEC-Standard-Users membership not reviewed | Add quarterly standard user review | Medium |
| No Sentinel alert on review completion | Review outcomes not fed to SIEM | Configure audit log export to Sentinel — Week 3 | Low |

---

## 9. Compliance Alignment

| Control | PCI-DSS | PIPEDA | NIST CSF | Zero Trust |
|---|---|---|---|---|
| Periodic access revalidation (AR-01) | Req 7.2.4, 8.6 | Principle 4 | PR.AC-1 | Least privilege |
| High-risk group monthly review (AR-02) | Req 7.2.4, 8.2 | Principle 4 | PR.AC-4 | Assume breach |
| Auto-remove on no response | Req 7.2 | Principle 4 | PR.AC-1 | Least privilege |
| Justification required | Req 10.2 | Principle 4 | DE.CM-3 | Assume breach |
| Decision helpers — inactive accounts | Req 8.2.6 | Principle 4 | PR.AC-1 | Least privilege |
| Full audit trail | Req 10.2, 10.3 | Principle 4 | DE.CM-3 | Assume breach |

---

## 10. Evidence Index

| File | Description |
|---|---|
| [P3-AR-00-Access-Reviews-List.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-00-Access-Reviews-List.png) | Both access reviews active in list |
| [P3-AR-01-Privileged-Groups-Review-Settings.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-01-Privileged-Groups-Review-Settings.png) | AR-01 full configuration — quarterly privileged groups |
| [P3-AR-01b-Privileged-Groups-Review-Created.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-01b-Privileged-Groups-Review-Created.png) | AR-01 created and active |
| [P3-AR-02-CA-Exclusions-Review-Settings.png](../Evidence/Phase-01-Identity/Access-Reviews/P3-AR-02-CA-Exclusions-Review-Settings.png) | AR-02 full configuration — monthly CA exclusions |

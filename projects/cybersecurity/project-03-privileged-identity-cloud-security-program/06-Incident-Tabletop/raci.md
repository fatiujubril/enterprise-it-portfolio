# Incident Response RACI
## FatiuLab Security — Identity Compromise Incident Response

**Incident Type:** Identity Compromise + Cloud Privilege Abuse  
**Reference Scenario:** INC-2026-001  
**Last Reviewed:** May 2026

---

## 1. RACI Key

| Letter | Meaning |
|---|---|
| R | Responsible — does the work |
| A | Accountable — owns the outcome, approves decisions |
| C | Consulted — provides input before action |
| I | Informed — notified of actions and outcomes |

---

## 2. Roles

| Role | Person | Responsibilities |
|---|---|---|
| Global Administrator | globaladmin@ | Tenant owner, final authority on all identity and Azure decisions |
| Security Administrator | securityadmin@ | Security operations, detection triage, investigation |
| Break-Glass | breakglass@ | Emergency access only — used when all other accounts unavailable |

---

## 3. RACI Matrix — Incident Response Activities

### Phase 1 — Detection & Triage

| Activity | Global Admin | Security Admin | Break-Glass |
|---|---|---|---|
| Monitor Sentinel dashboard | I | R | — |
| Triage Sentinel alerts | C | R | — |
| Declare P1 incident | A | R | — |
| Notify Global Admin of P1 | I | R | — |
| Review correlated alerts | R | R | — |
| Determine incident scope | A | R | — |

---

### Phase 2 — Containment

| Activity | Global Admin | Security Admin | Break-Glass |
|---|---|---|---|
| Disable compromised account | R | C | — |
| Revoke active sessions | R | C | — |
| Re-enable disabled CA policies | R | C | — |
| Deny malicious PIM requests | R | I | — |
| Remove Azure role assignments | R | C | — |
| Deactivate active PIM roles | R | C | — |
| Isolate affected resources | R | C | — |
| Use break-glass if GA compromised | A | R | R |

---

### Phase 3 — Investigation

| Activity | Global Admin | Security Admin | Break-Glass |
|---|---|---|---|
| Pull Sentinel incident logs | C | R | — |
| Review Entra ID audit logs | C | R | — |
| Review Azure Activity logs | C | R | — |
| Establish attack timeline | I | R | — |
| Identify initial access vector | I | R | — |
| Determine blast radius | A | R | — |
| Document findings | I | R | — |

---

### Phase 4 — Eradication

| Activity | Global Admin | Security Admin | Break-Glass |
|---|---|---|---|
| Delete attacker-deployed resources | R | C | — |
| Remove attacker-created accounts | R | C | — |
| Reset compromised account credentials | R | C | — |
| Verify no persistent backdoors | C | R | — |
| Confirm CA policies in correct state | R | C | — |
| Confirm PIM settings unchanged | R | C | — |

---

### Phase 5 — Recovery

| Activity | Global Admin | Security Admin | Break-Glass |
|---|---|---|---|
| Create replacement admin account | R | I | — |
| Enroll MFA on new account | R | R | — |
| Restore PIM eligible assignments | R | C | — |
| Verify all detection rules active | C | R | — |
| Confirm Secure Score baseline | C | R | — |
| Declare incident resolved | A | C | — |

---

### Phase 6 — Post-Incident

| Activity | Global Admin | Security Admin | Break-Glass |
|---|---|---|---|
| Write incident report | C | R | — |
| Conduct lessons learned | A | R | — |
| Update risk register | A | R | — |
| Implement backlog improvements | A | R | — |
| Brief stakeholders | R | C | — |
| Update exception register if needed | A | R | — |

---

## 4. Escalation Matrix

| Scenario | Action | Who |
|---|---|---|
| Sentinel alert fires — single rule | Triage within 30 minutes | Security Admin |
| Two correlated alerts | Declare P1, notify Global Admin | Security Admin |
| Global Admin account suspected compromised | Use break-glass account | Break-Glass |
| Break-glass account used | Immediate notification, document use | Global Admin |
| PIM unavailable | Use break-glass for emergency access | Break-Glass |
| All admin accounts locked out | Use break-glass credentials from offline storage | Break-Glass |

---

## 5. Communication Plan

| Audience | What to Communicate | When | Who |
|---|---|---|---|
| Security Admin | Alert details, triage request | Immediately on P1 | Sentinel (automated) |
| Global Admin | P1 declared, action required | Within 5 minutes | Security Admin |
| Executive team | Incident summary, business impact | Within 1 hour | Global Admin |
| Affected users | Account reset, re-enrollment required | During recovery | Global Admin |
| Regulators (if data breach) | Breach notification per PIPEDA | Within 72 hours | Global Admin |

# MITRE ATT&CK Coverage
## FatiuLab Security — Detection Engineering Coverage Matrix

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Framework:** MITRE ATT&CK for Enterprise v14  
**Tenant:** FatiuLabSecurity.onmicrosoft.com  
**Last Reviewed:** May 2026

---

## 1. Coverage Summary

| Metric | Value |
|---|---|
| Total detection rules | 8 |
| MITRE Tactics covered | 3 |
| MITRE Techniques covered | 4 |
| MITRE Sub-techniques covered | 6 |
| Data sources | 3 (AuditLogs, SigninLogs, AzureActivity) |

---

## 2. Tactics Coverage

| Tactic | ID | Rules Covering | Coverage |
|---|---|---|---|
| Initial Access | TA0001 | DET-05, DET-07, DET-08 | 3 rules |
| Privilege Escalation | TA0004 | DET-01, DET-02, DET-06 | 3 rules |
| Defense Evasion | TA0005 | DET-03, DET-04 | 2 rules |

---

## 3. Technique Coverage Matrix

| Technique | Sub-technique | Name | Rules | Data Source |
|---|---|---|---|---|
| T1078 | T1078.004 | Valid Accounts — Cloud Accounts | DET-01, DET-02, DET-05, DET-07 | AuditLogs, SigninLogs |
| T1098 | T1098.003 | Account Manipulation — Additional Cloud Roles | DET-06 | AzureActivity |
| T1190 | — | Exploit Public-Facing Application | DET-08 | AzureActivity |
| T1556 | T1556.006 | Modify Authentication — MFA | DET-03 | AuditLogs |
| T1556 | T1556.009 | Modify Authentication — Conditional Access | DET-04 | AuditLogs |

---

## 4. Rule-to-MITRE Mapping

| Rule | Tactic | Technique | Sub-technique | Rationale |
|---|---|---|---|---|
| DET-01 PIM Outside Hours | Privilege Escalation | T1078 | T1078.004 | Attacker using stolen cloud credentials to activate privileged roles outside normal hours |
| DET-02 GlobalAdmin Outside PIM | Privilege Escalation | T1078 | T1078.004 | Attacker bypassing JIT controls to establish persistent privileged cloud account access |
| DET-03 MFA Disabled | Defense Evasion | T1556 | T1556.006 | Attacker modifying MFA configuration to remove authentication barrier |
| DET-04 CA Policy Modified | Defense Evasion | T1556 | T1556.009 | Attacker weakening Conditional Access policies to bypass authentication controls |
| DET-05 Impossible Travel | Initial Access | T1078 | T1078.004 | Attacker using stolen credentials from different geographic location |
| DET-06 Azure Role Assignment | Privilege Escalation | T1098 | T1098.003 | Attacker adding themselves or a controlled account to privileged Azure roles |
| DET-07 Break-Glass Sign-In | Initial Access | T1078 | T1078.004 | Attacker using stolen break-glass credentials — bypasses all CA controls |
| DET-08 Defender Finding | Initial Access | T1190 | — | Active exploitation of exposed or misconfigured cloud resource |

---

## 5. Attack Chain Coverage

These rules are designed to detect attacks at multiple stages of the identity attack chain:

```
RECONNAISSANCE          Not covered — pre-attack phase
        ↓
INITIAL ACCESS          DET-05 Impossible Travel
                        DET-07 Break-Glass Sign-In
                        DET-08 Defender Finding
        ↓
EXECUTION               Not covered — focus is identity layer
        ↓
PERSISTENCE             DET-02 GlobalAdmin Outside PIM
                        DET-06 Azure Role Assignment
        ↓
PRIVILEGE ESCALATION    DET-01 PIM Outside Hours
                        DET-02 GlobalAdmin Outside PIM
                        DET-06 Azure Role Assignment
        ↓
DEFENSE EVASION         DET-03 MFA Disabled
                        DET-04 CA Policy Modified
        ↓
LATERAL MOVEMENT        Not covered — future roadmap
        ↓
COLLECTION/EXFIL        Not covered — future roadmap
```

**Coverage focus:** The detection set is intentionally concentrated on Initial Access, Privilege Escalation, and Defense Evasion — the three tactics most relevant to identity-based attacks targeting a FinTech tenant.

---

## 6. Identity Attack Scenario Coverage

### Scenario 1 — Credential Theft + Privilege Escalation
```
Attacker obtains admin credentials via phishing
        ↓
Signs in from foreign IP → DET-05 fires (Impossible Travel)
        ↓
Activates PIM role at 2am → DET-01 fires (Outside Business Hours)
        ↓
Assigns themselves Owner role in Azure → DET-06 fires (Role Assignment)
```
**Coverage: 3 rules across the full attack chain** ✅

### Scenario 2 — Insider Threat / Compromised Admin
```
Malicious insider or compromised admin account
        ↓
Disables MFA for target account → DET-03 fires (MFA Disabled)
        ↓
Modifies CA policy to remove MFA requirement → DET-04 fires (CA Modified)
        ↓
Assigns Global Admin directly outside PIM → DET-02 fires (GA Outside PIM)
```
**Coverage: 3 rules detecting defense weakening before escalation** ✅

### Scenario 3 — Break-Glass Account Compromise
```
Attacker discovers break-glass credentials
(stored in unprotected location, or obtained via social engineering)
        ↓
Signs in as break-glass → DET-07 fires immediately (NRT — 1-2 min)
        ↓
Full Global Admin access, all CA policies bypassed
```
**Coverage: Immediate NRT detection** ✅

---

## 7. Coverage Gaps — Future Roadmap

| Gap | MITRE Technique | Planned Rule | Priority |
|---|---|---|---|
| Lateral movement via service principals | T1550.001 | DET-09: Service principal sign-in anomaly | High |
| Data exfiltration from storage | T1530 | DET-10: Unusual storage account access | High |
| Password spray attacks | T1110.003 | DET-11: Multiple failed sign-ins across accounts | Medium |
| New admin user created | T1136.003 | DET-12: New cloud admin account created | Medium |
| Suspicious app registration | T1550.001 | DET-13: New OAuth app with high permissions | Medium |
| PIM alert disabled | T1562.001 | DET-14: PIM alert configuration modified | Low |

---

## 8. Data Source Coverage

| Data Source | Table | Events Ingested | Rules Using |
|---|---|---|---|
| Microsoft Entra ID | AuditLogs | Role management, policy changes, MFA events | DET-01, DET-02, DET-03, DET-04 |
| Microsoft Entra ID | SigninLogs | All authentication events | DET-05, DET-07 |
| Azure Activity | AzureActivity | Subscription-level operations, security alerts | DET-06, DET-08 |

Evidence: [P3-SEN-03-Data-Connectors.png](../Evidence/Phase-03-Detections/Sentinel/P3-SEN-03-Data-Connectors.png)

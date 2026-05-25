# Risk Register
## FatiuLab Security — Identity & Cloud Security Risk Register

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Regulatory Context:** PCI-DSS, PIPEDA  
**Risk Framework:** NIST CSF  
**Register Owner:** Global Administrator  
**Last Reviewed:** May 2026  
**Review Frequency:** Quarterly

---

## 1. Overview

This risk register documents the identified identity and cloud security risks for FatiuLab Security's Azure and Microsoft 365 environment. Each risk is assessed for likelihood and impact, assigned an owner, and mapped to the controls implemented as part of the Privileged Identity & Cloud Security Program.

**Risk scoring methodology:**

| Score | Likelihood | Impact |
|---|---|---|
| 1 | Rare — unlikely to occur | Minimal — negligible business impact |
| 2 | Unlikely — could occur occasionally | Minor — limited impact, easily recoverable |
| 3 | Possible — might occur at some point | Moderate — noticeable impact, recoverable |
| 4 | Likely — will probably occur | Significant — major impact, difficult recovery |
| 5 | Almost certain — expected to occur | Critical — catastrophic impact, existential threat |

**Risk rating = Likelihood × Impact**

| Rating | Score | Action |
|---|---|---|
| Critical | 20-25 | Immediate remediation required |
| High | 12-19 | Remediation within 30 days |
| Medium | 6-11 | Remediation within 90 days |
| Low | 1-5 | Accept or remediate within 180 days |

---

## 2. Risk Register

---

### RISK-01 — Compromised Global Administrator Account

**Category:** Identity  
**NIST CSF Function:** Protect (PR.AC)

| Field | Value |
|---|---|
| Risk description | A Global Administrator account is compromised via phishing, credential stuffing, or password spray. Attacker gains full control of the tenant including all identity, security, and payment infrastructure. |
| Threat actor | External attacker, malicious insider |
| Likelihood | 3 — Possible |
| Impact | 5 — Critical |
| **Risk Rating** | **15 — High** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 15 — High

**Controls implemented:**
- CA01: MFA required for all users ← reduces likelihood
- CA02: MFA required for admins ← reduces likelihood
- PIM: JIT activation with approval — no standing privilege ← reduces impact
- CA03: Legacy auth blocked ← reduces likelihood
- DET-01, DET-02: PIM and GA assignment detection ← reduces impact (faster detection)

**Residual risk (with controls):** Likelihood 2, Impact 4 = **8 — Medium**

**Residual risk rationale:** MFA and PIM significantly reduce the likelihood of successful compromise and limit the time window of access if compromised. Detection rules reduce dwell time.

---

### RISK-02 — Break-Glass Account Compromise

**Category:** Identity  
**NIST CSF Function:** Protect (PR.AC), Detect (DE.CM)

| Field | Value |
|---|---|
| Risk description | The break-glass account credentials are discovered by an attacker. Since the account has no MFA and is excluded from all CA policies, the attacker gains immediate unrestricted Global Administrator access. |
| Threat actor | External attacker, insider threat |
| Likelihood | 2 — Unlikely |
| Impact | 5 — Critical |
| **Risk Rating** | **10 — Medium** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 10 — Medium

**Controls implemented:**
- Credentials stored offline — reduces likelihood of discovery
- Monthly access review (AR-02) — monitors SEC-CA-Exclusions group
- DET-07 (NRT): Break-glass sign-in detection — immediate alert
- No MFA registered — intentional design, not a control gap

**Residual risk (with controls):** Likelihood 1, Impact 5 = **5 — Low**

**Residual risk rationale:** Offline credential storage and NRT detection significantly reduce both likelihood and detection time. Any use triggers immediate alert within 1-2 minutes.

---

### RISK-03 — Privilege Escalation via PIM Bypass

**Category:** Identity  
**NIST CSF Function:** Protect (PR.AC)

| Field | Value |
|---|---|
| Risk description | An attacker with sufficient privileges directly assigns Global Administrator role to a controlled account, bypassing the PIM JIT workflow entirely. This creates persistent privileged access without an audit trail. |
| Threat actor | External attacker with existing privileged access, malicious insider |
| Likelihood | 2 — Unlikely |
| Impact | 5 — Critical |
| **Risk Rating** | **10 — Medium** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 15 — High

**Controls implemented:**
- PIM: No permanent active assignments except Global Admin safety anchor
- PIM settings: MFA + justification required even on direct active assignments
- DET-02: Global Admin assigned outside PIM — detection rule
- Quarterly access review (AR-01) — catches residual assignments

**Residual risk (with controls):** Likelihood 2, Impact 4 = **8 — Medium**

---

### RISK-04 — Conditional Access Policy Misconfiguration or Bypass

**Category:** Identity  
**NIST CSF Function:** Protect (PR.AC)

| Field | Value |
|---|---|
| Risk description | A CA policy is misconfigured or intentionally modified by an attacker, creating authentication bypass paths. For example, disabling the legacy auth block re-enables password spray attack vectors. |
| Threat actor | External attacker, misconfiguration |
| Likelihood | 2 — Unlikely |
| Impact | 4 — Significant |
| **Risk Rating** | **8 — Medium** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 12 — High

**Controls implemented:**
- CA03: Legacy auth blocked on all users — primary protection
- CA policy set: 5 policies covering all access scenarios
- DET-04: CA policy modification detection
- SEC-CA-Exclusions: Only break-glass account excluded

**Residual risk (with controls):** Likelihood 1, Impact 4 = **4 — Low**

---

### RISK-05 — MFA Authentication Bypass

**Category:** Identity  
**NIST CSF Function:** Protect (PR.AC)

| Field | Value |
|---|---|
| Risk description | An attacker bypasses MFA through SIM swapping, MFA fatigue attacks (repeated push notifications), or by disabling MFA on a target account before attempting access. |
| Threat actor | External attacker |
| Likelihood | 3 — Possible |
| Impact | 4 — Significant |
| **Risk Rating** | **12 — High** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 20 — Critical

**Controls implemented:**
- MFA enforced via CA01/CA02 for all users and admins
- DET-03: MFA disabled detection
- PIM: Additional MFA challenge on role activation

**Residual risk (with controls):** Likelihood 2, Impact 3 = **6 — Medium**

**Gap:** Microsoft Authenticator number matching enabled by default in new tenants — reduces MFA fatigue risk. Phishing-resistant MFA (FIDO2) not yet deployed — planned improvement.

---

### RISK-06 — Unauthorized Azure Resource Deployment

**Category:** Cloud  
**NIST CSF Function:** Protect (PR.AC), Detect (DE.CM)

| Field | Value |
|---|---|
| Risk description | An attacker with Owner or Contributor access deploys malicious Azure resources — crypto miners, command and control infrastructure, or data exfiltration endpoints — using the company's Azure subscription. |
| Threat actor | External attacker with Azure access |
| Likelihood | 2 — Unlikely |
| Impact | 4 — Significant |
| **Risk Rating** | **8 — Medium** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 12 — High

**Controls implemented:**
- CA04: MFA required for Azure Management access
- DET-06: Azure privileged role assignment detection
- Defender for Cloud CSPM: continuous posture monitoring
- CIS benchmark: compliance monitoring

**Residual risk (with controls):** Likelihood 1, Impact 3 = **3 — Low**

---

### RISK-07 — Storage Account Data Exposure

**Category:** Cloud  
**NIST CSF Function:** Protect (PR.DS)

| Field | Value |
|---|---|
| Risk description | Sensitive data stored in Azure Blob Storage is exposed due to misconfiguration — public access enabled, shared key access not disabled, or no network restrictions. |
| Threat actor | External attacker, accidental exposure |
| Likelihood | 3 — Possible |
| Impact | 4 — Significant |
| **Risk Rating** | **12 — High** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 20 — Critical

**Controls implemented:**
- Public network access disabled on fatiulabstorage01 ✅
- Shared key access disabled ✅
- Blob anonymous access disabled ✅
- Secure transfer required ✅
- Defender CSPM: continuous misconfiguration detection

**Residual risk (with controls):** Likelihood 1, Impact 3 = **3 — Low**

**Accepted risk:** Private endpoint and infrastructure encryption not implemented — accepted risk documented in exemptions register.

---

### RISK-08 — Privileged Access Accumulation

**Category:** Identity  
**NIST CSF Function:** Protect (PR.AC)

| Field | Value |
|---|---|
| Risk description | Over time users accumulate more access than their role requires — role changes without access removal, group memberships never reviewed, PIM eligibility never expired. This creates a large attack surface of over-privileged accounts. |
| Threat actor | Insider threat, compromised account |
| Likelihood | 4 — Likely (without governance controls) |
| Impact | 3 — Moderate |
| **Risk Rating** | **12 — High** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 12 — High

**Controls implemented:**
- PIM: Eligible assignments expire (3-6 months)
- AR-01: Quarterly privileged group review — auto-remove on no response
- AR-02: Monthly CA exclusions review
- Identity lifecycle process: defined joiner/mover/leaver procedures

**Residual risk (with controls):** Likelihood 2, Impact 3 = **6 — Medium**

---

### RISK-09 — Insufficient Audit Logging

**Category:** Cloud  
**NIST CSF Function:** Detect (DE.CM)

| Field | Value |
|---|---|
| Risk description | Security events are not captured in logs, or logs are not retained long enough for forensic investigation. An attacker could operate undetected for extended periods if key events are not logged. |
| Threat actor | External attacker, insider threat |
| Likelihood | 2 — Unlikely |
| Impact | 4 — Significant |
| **Risk Rating** | **8 — Medium** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 16 — High

**Controls implemented:**
- Sentinel: 11 data connectors streaming logs
- AuditLogs: Entra ID audit events captured
- SigninLogs: All authentication events captured
- AzureActivity: All Azure subscription events captured
- 8 detection rules monitoring key events

**Residual risk (with controls):** Likelihood 1, Impact 3 = **3 — Low**

**Gap:** Diagnostic settings not yet deployed on all resources (CIS Category 5 finding) — planned Week 3 remediation.

---

### RISK-10 — Regulatory Non-Compliance

**Category:** Compliance  
**NIST CSF Function:** Identify (ID.GV)

| Field | Value |
|---|---|
| Risk description | FatiuLab Security fails to meet PCI-DSS or PIPEDA requirements, resulting in regulatory penalties, loss of payment processing license, or reputational damage. |
| Threat actor | Regulatory audit, data breach |
| Likelihood | 2 — Unlikely |
| Impact | 5 — Critical |
| **Risk Rating** | **10 — Medium** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 15 — High

**Controls implemented:**
- CIS Azure Foundations Benchmark v2.0.0: 66/81 controls passing (81%)
- MCSB: 61/63 controls passing (97%)
- Quarterly access reviews — PCI-DSS Req 7.2.4
- MFA enforcement — PCI-DSS Req 8.4
- Full audit trail — PCI-DSS Req 10

**Residual risk (with controls):** Likelihood 1, Impact 4 = **4 — Low**

---

### RISK-11 — Insider Threat — Privileged User

**Category:** Identity  
**NIST CSF Function:** Detect (DE.CM), Protect (PR.AC)

| Field | Value |
|---|---|
| Risk description | A current employee with privileged access intentionally misuses their access — exfiltrating payment data, sabotaging infrastructure, or creating backdoor accounts. |
| Threat actor | Malicious insider |
| Likelihood | 2 — Unlikely |
| Impact | 5 — Critical |
| **Risk Rating** | **10 — Medium** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 15 — High

**Controls implemented:**
- PIM: JIT access — no standing privilege, limits exposure window
- All PIM activations require justification — audit trail
- DET-01: After-hours activation detection
- Access reviews: Quarterly revalidation of all privileged access
- CA policies: All admin activity requires MFA

**Residual risk (with controls):** Likelihood 1, Impact 4 = **4 — Low**

---

### RISK-12 — Cloud Security Posture Drift

**Category:** Cloud  
**NIST CSF Function:** Identify (ID.RA)

| Field | Value |
|---|---|
| Risk description | Over time, the security configuration of the Azure environment degrades — new resources deployed without security controls, existing controls disabled, benchmark compliance score declining. |
| Threat actor | Misconfiguration, uncontrolled change |
| Likelihood | 3 — Possible |
| Impact | 3 — Moderate |
| **Risk Rating** | **9 — Medium** |
| Risk owner | Global Administrator |

**Inherent risk (without controls):** 12 — High

**Controls implemented:**
- Defender for Cloud CSPM: continuous posture assessment
- CIS benchmark: compliance drift monitoring
- Secure Score tracking: 67% baseline established
- DET-08: High severity finding detection

**Residual risk (with controls):** Likelihood 2, Impact 2 = **4 — Low**

---

## 3. Risk Summary Matrix

| Risk ID | Risk Name | Inherent | Residual | Status |
|---|---|---|---|---|
| RISK-01 | Compromised Global Admin | High (15) | **Medium (8)** | Controlled |
| RISK-02 | Break-Glass Compromise | Medium (10) | **Low (5)** | Controlled |
| RISK-03 | PIM Bypass | High (15) | **Medium (8)** | Controlled |
| RISK-04 | CA Policy Bypass | High (12) | **Low (4)** | Controlled |
| RISK-05 | MFA Bypass | Critical (20) | **Medium (6)** | Controlled |
| RISK-06 | Unauthorized Azure Deployment | High (12) | **Low (3)** | Controlled |
| RISK-07 | Storage Data Exposure | Critical (20) | **Low (3)** | Controlled |
| RISK-08 | Privilege Accumulation | High (12) | **Medium (6)** | Controlled |
| RISK-09 | Insufficient Logging | High (16) | **Low (3)** | Controlled |
| RISK-10 | Regulatory Non-Compliance | High (15) | **Low (4)** | Controlled |
| RISK-11 | Insider Threat | High (15) | **Low (4)** | Controlled |
| RISK-12 | Posture Drift | High (12) | **Low (4)** | Controlled |

**No residual risks remain at High or Critical.** All risks have been reduced to Medium or Low through the controls implemented in this program.

---

## 4. NIST CSF Mapping

| NIST CSF Function | Category | Risks Addressed |
|---|---|---|
| Identify (ID) | ID.GV — Governance | RISK-10 |
| Identify (ID) | ID.RA — Risk Assessment | RISK-12 |
| Protect (PR) | PR.AC — Access Control | RISK-01, RISK-02, RISK-03, RISK-04, RISK-05, RISK-06, RISK-08, RISK-11 |
| Protect (PR) | PR.DS — Data Security | RISK-07 |
| Detect (DE) | DE.CM — Continuous Monitoring | RISK-02, RISK-09, RISK-11, RISK-12 |
| Respond (RS) | RS.CO — Communications | RISK-10 |

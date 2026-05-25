# Solution Architecture
## FatiuLab Security — Identity & Cloud Security Program Architecture

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Last Updated:** May 2026

---

## 1. Architecture Overview

The FatiuLab Security Identity & Cloud Security Program is built on a Microsoft-native architecture spanning four interconnected layers:

```
┌─────────────────────────────────────────────────────────────────┐
│                    IDENTITY GOVERNANCE LAYER                     │
│  Entra ID P2 · Conditional Access · PIM · Access Reviews        │
│  Who can access what, when, and under what conditions            │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                  CLOUD SECURITY POSTURE LAYER                    │
│  Defender for Cloud · Azure Policy · CIS Benchmark              │
│  What is deployed, how is it configured, is it compliant        │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                  DETECTION & RESPONSE LAYER                      │
│  Microsoft Sentinel · KQL Analytics Rules · MITRE ATT&CK        │
│  What is happening, is it malicious, how do we respond          │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                      GOVERNANCE LAYER                            │
│  Risk Register · NIST CSF · Exceptions · Tabletop               │
│  Is the program working, can we prove it, how do we improve     │
└─────────────────────────────────────────────────────────────────┘
```

Each layer feeds the next — identity governance events flow into detection, detection findings feed the risk register, and the risk register drives governance decisions.

---

## 2. Component Architecture

### 2.1 Identity Governance

```
Microsoft Entra ID P2
├── Users (4)
│   ├── Global Admin — permanent GA, primary operations
│   ├── Security Admin — PIM eligible only, JIT access
│   ├── Standard User — no admin role, CA policy testing
│   └── Break Glass — permanent GA, no MFA, emergency only
│
├── Security Groups (5)
│   ├── SEC-Global-Admins
│   ├── SEC-Admins
│   ├── SEC-Standard-Users
│   ├── SEC-CA-Exclusions ← break-glass only
│   └── SEC-PIM-Eligible
│
├── Conditional Access (5 policies)
│   ├── CA01 — MFA all users (On)
│   ├── CA02 — MFA admins (On)
│   ├── CA03 — Block legacy auth (On)
│   ├── CA04 — MFA Azure Management (On)
│   └── CA05 — Standard users MFA (Report-only)
│
├── Privileged Identity Management
│   ├── Global Administrator — 2hr, MFA, approval required
│   └── Security Administrator — 4hr, MFA, self-approve
│
└── Access Reviews
    ├── AR-01 — Quarterly privileged groups
    └── AR-02 — Monthly CA exclusions
```

### 2.2 Cloud Security Posture

```
Azure Subscription 1
├── Resource Group: RG-FatiuLab-Security
│   ├── fatiulabstorage01 (Storage Account)
│   │   ├── Public network access: Disabled
│   │   ├── Shared key access: Disabled
│   │   └── Secure transfer: Enabled
│   └── LAW-FatiuLab-Security (Log Analytics)
│
├── Defender for Cloud
│   ├── Foundational CSPM (Free — always on)
│   ├── Defender CSPM (Enabled)
│   └── CWPP plans (All off — no applicable resources)
│
└── Azure Policy
    ├── CIS Azure Foundations Benchmark v2.0.0
    ├── ASC Default (Microsoft Cloud Security Benchmark)
    └── Configure Azure Activity logs → Log Analytics
```

### 2.3 Detection & Response

```
Microsoft Sentinel (law-fatiulab-security)
├── Data Connectors (11)
│   ├── Microsoft Entra ID → AuditLogs, SigninLogs
│   ├── Azure Activity → AzureActivity
│   ├── Microsoft Defender for Cloud
│   └── Microsoft Defender XDR (+ related)
│
└── Analytics Rules (8)
    ├── DET-01 PIM Outside Hours (Scheduled)
    ├── DET-02 GlobalAdmin Outside PIM (Scheduled)
    ├── DET-03 MFA Disabled (Scheduled)
    ├── DET-04 CA Policy Modified (Scheduled)
    ├── DET-05 Impossible Travel (Scheduled)
    ├── DET-06 Azure Role Assignment (Scheduled)
    ├── DET-07 Break-Glass Sign-In (NRT)
    └── DET-08 Defender High Severity (Scheduled)
```

### 2.4 Governance

```
GRC Program
├── Risk Register (12 risks)
│   ├── All residual risks: Medium or Low
│   └── Framework: NIST CSF
│
├── Control Mapping
│   └── NIST CSF: ID, PR, DE, RS, RC all mapped
│
├── Exception Register (7 exceptions)
│   ├── EXP-01 Permanent GA
│   ├── EXP-02/03 Defender CWPP plans
│   ├── EXP-04/05/06 Storage controls
│   └── EXP-07 CA05 report-only
│
└── Incident Tabletop
    ├── Scenario: AiTM + Azure privilege abuse
    ├── 4 of 6 attack events detected
    └── 7-item improvement backlog
```

---

## 3. Data Flow Architecture

```
Authentication Event (user signs in)
        │
        ▼
Microsoft Entra ID
(evaluates CA policies)
        │
        ├── CA policies satisfied → Access granted
        │                           Event logged to SigninLogs
        │
        └── CA policies violated → Access denied
                                    Event logged to SigninLogs
                │
                ▼
        Microsoft Sentinel
        (SigninLogs ingested)
                │
                ▼
        Analytics Rules evaluate
        (DET-05 Impossible Travel,
         DET-07 Break-Glass)
                │
                ▼
        Incident created if threshold met
                │
                ▼
        Security team investigates
```

```
Azure Resource Operation (role assignment, policy change)
        │
        ▼
Azure Resource Manager
        │
        ▼
AzureActivity log generated
        │
        ▼
Microsoft Sentinel
(AzureActivity ingested)
        │
        ▼
Analytics Rules evaluate
(DET-06 Role Assignment,
 DET-08 Defender Finding)
        │
        ▼
Incident created if threshold met
```

---

## 4. Integration Points

| Integration | Source | Target | Purpose |
|---|---|---|---|
| Entra ID → Sentinel | AuditLogs, SigninLogs | LAW-FatiuLab-Security | Identity event detection |
| Azure Activity → Sentinel | AzureActivity | LAW-FatiuLab-Security | Cloud operation detection |
| Defender for Cloud → Sentinel | Security alerts | LAW-FatiuLab-Security | CSPM finding detection |
| Azure Policy → Defender for Cloud | CIS initiative | Regulatory compliance dashboard | Benchmark compliance tracking |
| PIM → AuditLogs | Role activations | AuditLogs table | JIT activation audit trail |
| CA policies → SigninLogs | Authentication decisions | SigninLogs table | Access control audit trail |

---

## 5. Security Boundaries

| Boundary | Control | Implementation |
|---|---|---|
| Authentication | MFA on every sign-in | CA01, CA02 |
| Privileged access | JIT only | PIM eligible assignments |
| Legacy protocols | Blocked entirely | CA03 |
| Azure Management | Separate MFA challenge | CA04 |
| Network access | Storage account isolated | Public network disabled |
| Emergency access | Break-glass with monitoring | SEC-CA-Exclusions + DET-07 |

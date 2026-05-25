# Detection Rules
## FatiuLab Security — Microsoft Sentinel Custom Analytics Rules

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**SIEM:** Microsoft Sentinel — workspace: law-fatiulab-security  
**Tenant:** FatiuLabSecurity.onmicrosoft.com  
**Rule Author:** Global Administrator  
**Last Reviewed:** May 2026  
**Status:** Active — 8 rules deployed

---

## 1. Overview

This document defines the 8 custom detection rules deployed in Microsoft Sentinel for FatiuLab Security. All rules are focused on identity and cloud security threats — the highest-risk attack surface for a FinTech handling payment data.

Rules are mapped to the MITRE ATT&CK framework and organized by data source. Each rule includes the full KQL query, trigger conditions, false positive guidance, and tuning notes.

**Detection philosophy:**

> Detection rules are not binary pass/fail checks. Every rule is a hypothesis: "If an attacker is doing X, we would expect to see Y in the logs." The goal is to detect attacker behavior with high fidelity — minimizing both missed detections (false negatives) and noise (false positives).

---

## 2. Rule Inventory

| ID | Name | Severity | Type | Data Source | MITRE Tactic | Technique |
|---|---|---|---|---|---|---|
| DET-01 | PIM Activation Outside Business Hours | Medium | Scheduled | AuditLogs | Privilege Escalation | T1078.004 |
| DET-02 | Global Admin Assigned Outside PIM | High | Scheduled | AuditLogs | Privilege Escalation | T1078.004 |
| DET-03 | MFA Disabled for Privileged Account | High | Scheduled | AuditLogs | Defense Evasion | T1556.006 |
| DET-04 | CA Policy Modified or Disabled | High | Scheduled | AuditLogs | Defense Evasion | T1556.009 |
| DET-05 | Impossible Travel on Admin Account | High | Scheduled | SigninLogs | Initial Access | T1078.004 |
| DET-06 | Azure Privileged Role Assignment | High | Scheduled | AzureActivity | Privilege Escalation | T1098.003 |
| DET-07 | Break-Glass Account Sign-In | High | NRT | SigninLogs | Initial Access | T1078.004 |
| DET-08 | Defender for Cloud High Severity Finding | High | Scheduled | AzureActivity | Initial Access | T1190 |

Evidence: [P3-DET-08-Defender-High-Severity.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-08-Defender-High-Severity.png)

---

## 3. Data Sources

### 3.1 Connected Data Connectors

| Connector | Tables | Rules Using It |
|---|---|---|
| Microsoft Entra ID | SigninLogs, AuditLogs | DET-01, DET-02, DET-03, DET-04, DET-05, DET-07 |
| Azure Activity | AzureActivity | DET-06, DET-08 |
| Tenant-based Microsoft Defender for Cloud | AzureActivity | DET-08 |

### 3.2 Log Ingestion Verification

All three data sources confirmed active before rule deployment:

```kql
-- AuditLogs: RoleManagement events confirmed
-- SigninLogs: Break-glass and admin sign-ins confirmed  
-- AzureActivity: Role assignment events confirmed
```

---

## 4. Detection Rules — Full Specification

---

### DET-01 — PIM Activation Outside Business Hours

**Hypothesis:** A legitimate user activates PIM roles during business hours (7am-7pm). Activation outside these hours may indicate a compromised account being used by an attacker in a different timezone, or an insider threat acting outside normal working patterns.

| Field | Value |
|---|---|
| Severity | Medium |
| Rule type | Scheduled |
| Run frequency | Every 1 hour |
| Lookup period | Last 1 hour |
| Data source | AuditLogs |
| MITRE Tactic | Privilege Escalation |
| MITRE Technique | T1078 — Valid Accounts |
| MITRE Sub-technique | T1078.004 — Cloud Accounts |

**KQL Query:**
```kql
AuditLogs
| where OperationName == "Add member to role in PIM completed (permanent)"
    or OperationName == "Add member to role in PIM completed (timebound)"
| extend InitiatedByUser = tostring(InitiatedBy.user.userPrincipalName)
| extend TargetUser = tostring(TargetResources[0].userPrincipalName)
| extend RoleName = tostring(TargetResources[0].displayName)
| extend ActivationHour = hourofday(TimeGenerated)
| where ActivationHour < 7 or ActivationHour > 19
| project TimeGenerated, InitiatedByUser, TargetUser, RoleName, ActivationHour, OperationName
```

**Trigger conditions:**
- PIM role activation completed (permanent or timebound)
- Activation hour is before 7am or after 7pm UTC

**Expected false positives:**
- Legitimate after-hours incident response
- Users in different timezones
- On-call security engineers working outside normal hours

**Tuning guidance:**
- Consider adding timezone offset if most users are in a specific region
- Whitelist known on-call accounts if after-hours activations are expected
- Cross-reference with change management tickets for planned after-hours work

Evidence: [P3-DET-01-PIM-Outside-Hours.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-01-PIM-Outside-Hours.png)

---

### DET-02 — Global Admin Assigned Outside PIM

**Hypothesis:** Global Administrator role should only ever be assigned through the PIM eligible → activation workflow. A direct assignment bypassing PIM indicates either a misconfiguration, an admin error, or an attacker with sufficient privileges attempting to establish persistent privileged access.

| Field | Value |
|---|---|
| Severity | High |
| Rule type | Scheduled |
| Run frequency | Every 1 hour |
| Lookup period | Last 1 hour |
| Data source | AuditLogs |
| MITRE Tactic | Privilege Escalation |
| MITRE Technique | T1078 — Valid Accounts |
| MITRE Sub-technique | T1078.004 — Cloud Accounts |

**KQL Query:**
```kql
AuditLogs
| where OperationName == "Add member to role"
| extend InitiatedByUser = tostring(InitiatedBy.user.userPrincipalName)
| extend TargetUser = tostring(TargetResources[0].userPrincipalName)
| extend RoleName = tostring(TargetResources[0].displayName)
| where RoleName == "Global Administrator"
| where not(OperationName has "PIM")
| project TimeGenerated, InitiatedByUser, TargetUser, RoleName, OperationName
```

**Trigger conditions:**
- "Add member to role" operation detected
- Target role is Global Administrator
- Operation does not contain "PIM" — indicates direct assignment not via JIT workflow

**Expected false positives:**
- Tenant creation automatically assigns first Global Admin — not applicable post-setup
- Break-glass account assignment — document as known exception

**Tuning guidance:**
- Add exclusion for break-glass account UPN if alert fires on legitimate BG assignment renewal
- Alert should fire very rarely — any trigger warrants immediate investigation

Evidence: [P3-DET-02-GlobalAdmin-Outside-PIM.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-02-GlobalAdmin-Outside-PIM.png)

---

### DET-03 — MFA Disabled for Privileged Account

**Hypothesis:** An attacker who has compromised a privileged account may attempt to disable MFA to remove an authentication barrier, making future logins easier and preventing the legitimate user from receiving MFA challenges that would alert them to unauthorized access.

| Field | Value |
|---|---|
| Severity | High |
| Rule type | Scheduled |
| Run frequency | Every 1 hour |
| Lookup period | Last 1 hour |
| Data source | AuditLogs |
| MITRE Tactic | Defense Evasion |
| MITRE Technique | T1556 — Modify Authentication Process |
| MITRE Sub-technique | T1556.006 — Multi-Factor Authentication |

**KQL Query:**
```kql
AuditLogs
| where OperationName in (
    "Delete user authentication method",
    "Update user",
    "Disable Strong Authentication",
    "Admin deleted MFA registration"
    )
| extend InitiatedByUser = tostring(InitiatedBy.user.userPrincipalName)
| extend TargetUser = tostring(TargetResources[0].userPrincipalName)
| extend ModifiedProperty = tostring(TargetResources[0].modifiedProperties[0].displayName)
| where InitiatedByUser != TargetUser
| project TimeGenerated, InitiatedByUser, TargetUser, OperationName, ModifiedProperty
```

**Trigger conditions:**
- MFA-related authentication method deleted or disabled
- Action initiated by a different user than the target (admin-initiated change)
- Covers multiple operation names to catch different MFA removal paths

**Expected false positives:**
- Helpdesk resetting MFA for a user who lost their device
- Admin removing old MFA methods during migration

**Tuning guidance:**
- Add helpdesk account exclusion if legitimate MFA resets generate noise
- Consider adding time correlation — MFA disable followed by sign-in within 1 hour is higher confidence

Evidence: [P3-DET-03-MFA-Disabled.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-03-MFA-Disabled.png)

---

### DET-04 — CA Policy Modified or Disabled

**Hypothesis:** Conditional Access policies are the primary authentication enforcement layer. An attacker with sufficient privileges may modify or disable CA policies to create authentication bypass paths — for example, disabling the legacy auth block to re-enable password spray attack vectors, or removing MFA requirements for specific users or apps.

| Field | Value |
|---|---|
| Severity | High |
| Rule type | Scheduled |
| Run frequency | Every 1 hour |
| Lookup period | Last 1 hour |
| Data source | AuditLogs |
| MITRE Tactic | Defense Evasion |
| MITRE Technique | T1556 — Modify Authentication Process |
| MITRE Sub-technique | T1556.009 — Conditional Access Policies |

**KQL Query:**
```kql
AuditLogs
| where OperationName in (
    "Update conditional access policy",
    "Delete conditional access policy",
    "Add conditional access policy"
    )
| extend InitiatedByUser = tostring(InitiatedBy.user.userPrincipalName)
| extend PolicyName = tostring(TargetResources[0].displayName)
| extend ModifiedProperties = tostring(TargetResources[0].modifiedProperties)
| project TimeGenerated, InitiatedByUser, PolicyName, OperationName, ModifiedProperties
```

**Trigger conditions:**
- CA policy created, updated, or deleted
- Covers all three operation types — creation of new policy could indicate attacker adding a permissive policy

**Expected false positives:**
- Legitimate CA policy updates by authorized admins
- Policy changes during planned maintenance windows

**Tuning guidance:**
- Every CA policy change should be correlated with a change management ticket
- Consider adding alert suppression during known maintenance windows
- ModifiedProperties field shows exactly what changed — use to triage severity of the change

Evidence: [P3-DET-04-CA-Policy-Modified.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-04-CA-Policy-Modified.png)

---

### DET-05 — Impossible Travel on Admin Account

**Hypothesis:** A user cannot physically be in two geographically distant locations within a short timeframe. Sign-ins from two different countries within 2 hours for the same admin account indicate either credential theft (attacker in a different country) or account sharing (policy violation).

| Field | Value |
|---|---|
| Severity | High |
| Rule type | Scheduled |
| Run frequency | Every 1 hour |
| Lookup period | Last 2 hours |
| Data source | SigninLogs |
| MITRE Tactic | Initial Access |
| MITRE Technique | T1078 — Valid Accounts |
| MITRE Sub-technique | T1078.004 — Cloud Accounts |

**KQL Query:**
```kql
SigninLogs
| where UserType == "Member"
| where ResultType == 0
| extend City = tostring(LocationDetails.city)
| extend Country = tostring(LocationDetails.countryOrRegion)
| extend Latitude = tostring(LocationDetails.geoCoordinates.latitude)
| extend Longitude = tostring(LocationDetails.geoCoordinates.longitude)
| project TimeGenerated, UserPrincipalName, IPAddress, City, Country, AppDisplayName
| join kind=inner (
    SigninLogs
    | where UserType == "Member"
    | where ResultType == 0
    | extend City2 = tostring(LocationDetails.city)
    | extend Country2 = tostring(LocationDetails.countryOrRegion)
    | project TimeGenerated2 = TimeGenerated, UserPrincipalName, IPAddress2 = IPAddress, City2, Country2
) on UserPrincipalName
| where Country != Country2
| where abs(datetime_diff('minute', TimeGenerated, TimeGenerated2)) < 120
| where TimeGenerated > TimeGenerated2
| project TimeGenerated, UserPrincipalName, IPAddress, City, Country, IPAddress2, City2, Country2, TimeGenerated2
```

**Trigger conditions:**
- Two successful sign-ins (ResultType == 0) from same user
- Sign-ins from different countries
- Time between sign-ins less than 120 minutes

**Expected false positives:**
- VPN usage — user appears in VPN exit country not physical location
- Corporate proxy routing traffic through different country
- Geolocation database inaccuracies

**Tuning guidance:**
- VPN exit nodes are the primary false positive source — consider maintaining a VPN IP allowlist
- Reduce false positives by increasing the travel time threshold or adding distance calculation
- Cross-reference with known VPN ranges if noise is high

Evidence: [P3-DET-05-Impossible-Travel.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-05-Impossible-Travel.png)

---

### DET-06 — Azure Privileged Role Assignment

**Hypothesis:** Owner and Contributor roles at subscription scope grant broad control over all Azure resources — equivalent to root access in a cloud environment. Unexpected assignment of these roles may indicate privilege escalation by an attacker who has compromised a privileged account.

| Field | Value |
|---|---|
| Severity | High |
| Rule type | Scheduled |
| Run frequency | Every 1 hour |
| Lookup period | Last 1 hour |
| Data source | AzureActivity |
| MITRE Tactic | Privilege Escalation |
| MITRE Technique | T1098 — Account Manipulation |
| MITRE Sub-technique | T1098.003 — Additional Cloud Roles |

**KQL Query:**
```kql
AzureActivity
| where OperationNameValue == "MICROSOFT.AUTHORIZATION/ROLEASSIGNMENTS/WRITE"
| where ActivityStatusValue == "Success"
| extend RoleAssignmentDetails = parse_json(Properties)
| extend RequestBody = tostring(RoleAssignmentDetails.requestbody)
| extend Caller = tostring(Caller)
| extend ResourceGroup = tostring(ResourceGroup)
| where RequestBody contains "Owner" or RequestBody contains "Contributor"
| project TimeGenerated, Caller, ResourceGroup, OperationNameValue, ActivityStatusValue, RequestBody
```

**Trigger conditions:**
- Azure role assignment write operation — successful
- Role assigned is Owner or Contributor
- Catches both subscription-level and resource group-level assignments

**Expected false positives:**
- Legitimate infrastructure deployments requiring temporary elevated access
- Automation service principals being granted access for deployment pipelines

**Tuning guidance:**
- Add exclusion for known deployment service principal accounts
- Consider scoping to subscription-level only to reduce noise from resource group assignments
- Any Owner assignment at subscription scope should always be investigated

Evidence: [P3-DET-06-Azure-Role-Assignment.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-06-Azure-Role-Assignment.png)

---

### DET-07 — Break-Glass Account Sign-In

**Hypothesis:** The break-glass account is an emergency-only account that should never be used during normal operations. Any sign-in by this account outside a declared emergency is a critical security event — it may indicate the account credentials have been compromised, or someone is using the account to bypass all CA policy controls.

| Field | Value |
|---|---|
| Severity | High |
| Rule type | NRT (Near Real Time) |
| Detection latency | ~1-2 minutes |
| Data source | SigninLogs |
| MITRE Tactic | Initial Access |
| MITRE Technique | T1078 — Valid Accounts |
| MITRE Sub-technique | T1078.004 — Cloud Accounts |

**Why NRT for this rule:**
Break-glass sign-ins require immediate awareness — not hourly awareness. A compromised break-glass account bypasses all Conditional Access policies and has permanent Global Administrator access. Every minute of delay in detection increases attacker dwell time in the highest-privilege account in the tenant.

**KQL Query:**
```kql
SigninLogs
| where UserPrincipalName contains "breakglass"
| extend City = tostring(LocationDetails.city)
| extend Country = tostring(LocationDetails.countryOrRegion)
| extend IPAddress = tostring(IPAddress)
| extend AppUsed = tostring(AppDisplayName)
| project TimeGenerated, UserPrincipalName, IPAddress, City, Country, AppUsed, ResultType, ResultDescription
```

**Trigger conditions:**
- Any sign-in event where UPN contains "breakglass"
- Triggers on both successful and failed sign-ins
- Failed sign-in attempts on break-glass account also warrant investigation

**Expected false positives:**
- Declared emergency use — Security Admin or Global Admin has explicitly communicated the emergency
- Quarterly break-glass credential test (best practice — verify credentials work before you need them)

**Tuning guidance:**
- This rule should essentially never fire in normal operations
- Do not suppress or tune down — every trigger is a high-priority investigation
- Consider adding automation playbook: auto-create P1 incident, notify Security Admin via email

Evidence: [P3-DET-07-Break-Glass-Sign-In.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-07-Break-Glass-Sign-In.png)

---

### DET-08 — Defender for Cloud High Severity Finding

**Hypothesis:** A high or critical severity finding from Defender for Cloud indicates either an active threat or a critical misconfiguration that creates significant attack surface. These findings require immediate triage to determine if they represent active exploitation or urgent remediation needs.

| Field | Value |
|---|---|
| Severity | High |
| Rule type | Scheduled |
| Run frequency | Every 1 hour |
| Lookup period | Last 1 hour |
| Data source | AzureActivity |
| MITRE Tactic | Initial Access |
| MITRE Technique | T1190 — Exploit Public-Facing Application |

**KQL Query:**
```kql
AzureActivity
| where CategoryValue == "Security"
| where ActivityStatusValue == "Active"
| where OperationNameValue contains "Microsoft.Security"
| extend AlertSeverity = tostring(parse_json(Properties).severity)
| extend AlertName = tostring(parse_json(Properties).alertDisplayName)
| extend ResourceId = tostring(parse_json(Properties).compromisedEntity)
| where AlertSeverity in ("High", "Critical")
| project TimeGenerated, AlertName, AlertSeverity, ResourceId, OperationNameValue, Caller
```

**Trigger conditions:**
- Defender for Cloud security event in AzureActivity
- Alert severity is High or Critical
- Active status — not resolved or dismissed findings

**Expected false positives:**
- Defender for Cloud recommendations surfaced as alerts
- Known accepted risks that haven't been exempted yet

**Tuning guidance:**
- Exempt known accepted risks in Defender for Cloud to prevent repeated alerts
- Consider filtering out specific finding types that are consistently false positives
- Cross-reference with the exemptions register before investigating

Evidence: [P3-DET-08-Defender-High-Severity.png](../Evidence/Phase-03-Detections/Analytics-Rules/P3-DET-08-Defender-High-Severity.png)

---

## 5. Rule Type Decision Matrix

| Rule | Why Scheduled / NRT |
|---|---|
| DET-01 PIM Outside Hours | Scheduled — requires hourofday() time calculation |
| DET-02 GlobalAdmin Outside PIM | Scheduled — 1hr frequency acceptable for role assignment detection |
| DET-03 MFA Disabled | Scheduled — 1hr frequency acceptable |
| DET-04 CA Policy Modified | Scheduled — 1hr frequency acceptable |
| DET-05 Impossible Travel | Scheduled — requires join across two time windows |
| DET-06 Azure Role Assignment | Scheduled — 1hr frequency acceptable |
| DET-07 Break-Glass Sign-In | **NRT** — immediate detection required, simple query |
| DET-08 Defender High Severity | Scheduled — aggregation and parsing required |

---

## 6. Sentinel Architecture

| Component | Value |
|---|---|
| Workspace | law-fatiulab-security |
| Workspace ID | e1030a74-426b-41b7-baf8-3cef510c9753 |
| Resource group | RG-FatiuLab-Security |
| Region | Canada Central |
| Data connectors | 11 connected |
| Active analytics rules | 8 |
| Rule types | 7 Scheduled + 1 NRT |

Evidence:
- [P3-SEN-01-LogAnalytics-Workspace.png](../Evidence/Phase-03-Detections/Sentinel/P3-SEN-01-LogAnalytics-Workspace.png)
- [P3-SEN-02-Sentinel-Workspace.png](../Evidence/Phase-03-Detections/Sentinel/P3-SEN-02-Sentinel-Workspace.png)
- [P3-SEN-03-Data-Connectors.png](../Evidence/Phase-03-Detections/Sentinel/P3-SEN-03-Data-Connectors.png)

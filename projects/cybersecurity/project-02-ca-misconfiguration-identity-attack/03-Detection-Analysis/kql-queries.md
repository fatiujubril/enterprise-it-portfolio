# KQL Queries - Conditional Access MFA Bypass Detection

Queries for detecting CA misconfigurations resulting in MFA bypass.
Designed for Microsoft Sentinel or Log Analytics with Entra ID sign-in logs.

---

## Query 1 - Successful Sign-Ins Without MFA

Identifies any user authenticating successfully without MFA.

```kql
SigninLogs
| where ResultType == 0
| where AuthenticationRequirement == "singleFactorAuthentication"
| project TimeGenerated,
          UserPrincipalName,
          AppDisplayName,
          ConditionalAccessStatus,
          AuthenticationRequirement,
          IPAddress,
          Location
| order by TimeGenerated desc
```

**MITRE:** T1078 - Valid Accounts
**Use case:** Baseline detection across all users.

---

## Query 2 - Conditional Access Not Applied

Detects sign-ins where CA evaluation did not occur.

```kql
SigninLogs
| where ResultType == 0
| where ConditionalAccessStatus == "notApplied"
| project TimeGenerated,
          UserPrincipalName,
          AppDisplayName,
          ConditionalAccessStatus,
          AuthenticationRequirement,
          IPAddress,
          Location
| order by TimeGenerated desc
```

**MITRE:** T1556 - Modify Authentication Process
**Use case:** Detects authentication events bypassing CA entirely.

---

## Query 3 - High-Value Account MFA Bypass

Focuses detection on executive and privileged identities.

```kql
let HighValueUsers = dynamic([
  "ceo@yourtenant.onmicrosoft.com",
  "cfo@yourtenant.onmicrosoft.com",
  "admin@yourtenant.onmicrosoft.com"
]);
SigninLogs
| where ResultType == 0
| where UserPrincipalName in (HighValueUsers)
| where AuthenticationRequirement == "singleFactorAuthentication"
| project TimeGenerated,
          UserPrincipalName,
          AppDisplayName,
          ConditionalAccessStatus,
          AuthenticationRequirement,
          IPAddress,
          Location
| order by TimeGenerated desc
```

**MITRE:** T1078 - Valid Accounts (Privileged)
**Use case:** Prioritized alerting for executive and admin accounts.

---

## Query 4 - MFA Enforcement Validation (Before vs After)

Validates remediation by comparing auth behavior over time.

```kql
SigninLogs
| where UserPrincipalName == "ceo@yourtenant.onmicrosoft.com"
| where ResultType == 0
| project TimeGenerated,
          AppDisplayName,
          ConditionalAccessStatus,
          AuthenticationRequirement
| order by TimeGenerated desc
```

**Use case:** Post-remediation validation - confirms MFA enforcement restored.

---

## Query 5 - CA Policy Exclusion Audit (Audit Logs)

Detects CA policy modifications that introduce exclusions.

```kql
AuditLogs
| where OperationName == "Update conditional access policy"
| extend ModifiedBy = tostring(InitiatedBy.user.userPrincipalName)
| extend PolicyName = tostring(TargetResources[0].displayName)
| project TimeGenerated, PolicyName, ModifiedBy, Result
| order by TimeGenerated desc
```

**Use case:** Proactive detection of CA policy changes that may introduce bypass risk.

---

## False Positive Exclusions

```kql
| where UserPrincipalName !in ("breakglass@yourtenant.onmicrosoft.com")
```

Always exclude break-glass accounts from MFA bypass alerts.

---

## Analyst Notes

- Policy presence does not guarantee enforcement - always validate via telemetry
- Auth outcome fields are more reliable than CA configuration review
- High-value identities should have lower alert thresholds
- Re-run Query 4 after any CA policy change to validate enforcement

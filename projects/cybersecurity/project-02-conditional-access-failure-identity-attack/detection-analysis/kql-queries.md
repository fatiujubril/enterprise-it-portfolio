# KQL Queries – Conditional Access MFA Bypass Detection

This document provides example Kusto Query Language (KQL) queries that can be used to detect Conditional Access misconfigurations resulting in multifactor authentication (MFA) bypass.

These queries are designed for use in Microsoft Entra ID sign-in logs ingested into a SIEM such as Microsoft Sentinel or Log Analytics.

---

## Query Objective

Identify successful interactive sign-ins where:
- Conditional Access was **not applied**
- Authentication was downgraded to **single-factor**
- The affected user is a high-value or business-critical identity

---

## Query 1 – Successful Sign-Ins Without MFA

This query identifies successful interactive sign-ins where MFA was not required.

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
**Use case:**
Baseline detection to identify any users authenticating without MFA.

## Query 2 – Conditional Access Not Applied

This query identifies successful sign-ins where Conditional Access evaluation did not occur.

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
**Use case:**
Detects authentication events that bypass Conditional Access entirely.

## Query 3 – High-Value Account MFA Bypass

This query focuses on business-critical identities by filtering for specific users.

```kql
let HighValueUsers = dynamic([
  "ceo@fatiulab.onmicrosoft.com"
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
**Use case:**
Prioritizes detection for executive or high-impact accounts where MFA bypass presents elevated risk.

## Query 4 – MFA Enforcement Validation (Before vs After)

This query validates remediation by comparing authentication behavior for a specific user over time.

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
**Use case:**
Confirms that MFA enforcement is restored after Conditional Access exceptions are removed.

## False Positive Considerations

The following account types may legitimately authenticate without MFA and should be excluded where appropriate:
- Break-glass emergency access accounts
- Service or automation accounts
- Approved, time-bound exception groups

Example exclusion:
```kql
| where UserPrincipalName !in ("breakglass@yourtenant.onmicrosoft.com")
```
## Analyst Notes

- Conditional Access policy presence does not guarantee enforcement
- Authentication outcome fields provide the most reliable signal
- High-value identities should be monitored with stricter alerting thresholds
- Detection logic should be reviewed after any Conditional Access policy change

## Outcome
These queries enable SOC analysts to reliably detect silent Conditional Access bypass conditions and validate MFA enforcement across high-risk identities using authentication telemetry rather than configuration assumptions.

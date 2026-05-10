# KQL Queries

Queries used to detect and investigate this incident in Microsoft Sentinel / Entra ID.

---

## Query 1 - Detect Anomalous Sign-Ins (Foreign IP, Single Factor)

```kql
SigninLogs
| where TimeGenerated > ago(7d)
| where AuthenticationRequirement == "singleFactorAuthentication"
| where RiskLevelDuringSignIn in ("medium", "high")
| project TimeGenerated, UserPrincipalName, IPAddress, Location,
          AuthenticationRequirement, ResultType, ResultDescription
| order by TimeGenerated desc
```

**Purpose:** Surfaces successful single-factor authentications with elevated risk scores.
**MITRE:** T1078 - Valid Accounts

---

## Query 2 - Detect Inbox Rule Creation After Sign-In Anomaly

```kql
OfficeActivity
| where TimeGenerated > ago(7d)
| where Operation == "New-InboxRule"
| project TimeGenerated, UserId, ClientIP, Parameters
| order by TimeGenerated desc
```

**Purpose:** Identifies inbox rule creation events for correlation with anomalous sign-ins.
**MITRE:** T1114.003 - Email Collection: Inbox Rule Manipulation

---

## Query 3 - Correlate Sign-In with Mailbox Rule Creation (Time Window Join)

```kql
let suspiciousSignins = SigninLogs
| where TimeGenerated > ago(7d)
| where AuthenticationRequirement == "singleFactorAuthentication"
| where RiskLevelDuringSignIn in ("medium", "high")
| project SigninTime = TimeGenerated, UserPrincipalName, IPAddress;
OfficeActivity
| where TimeGenerated > ago(7d)
| where Operation == "New-InboxRule"
| project RuleTime = TimeGenerated, UserId, Parameters
| join kind=inner suspiciousSignins on $left.UserId == $right.UserPrincipalName
| where RuleTime between (SigninTime .. (SigninTime + 1h))
| project SigninTime, RuleTime, UserId, IPAddress, Parameters
```

**Purpose:** Confirms inbox rule was created by the same session as the anomalous sign-in.
**MITRE:** T1078 + T1114.003 correlation

---

## Query 4 - Detect Outbound Email Spike from Compromised Mailbox

```kql
EmailEvents
| where TimeGenerated > ago(7d)
| where SenderFromAddress == "compromised-user@domain.com"
| where EmailDirection == "Outbound"
| summarize EmailCount = count() by bin(TimeGenerated, 1h), SenderFromAddress
| order by TimeGenerated desc
```

**Purpose:** Identifies unusual outbound email volume from a specific sender.
**MITRE:** T1534 - Internal Spearphishing

---

## Notes

- Queries are written for Microsoft Sentinel using standard table schemas
- Adjust time ranges and thresholds based on your environment baseline
- Query 3 requires both SigninLogs and OfficeActivity connectors enabled in Sentinel

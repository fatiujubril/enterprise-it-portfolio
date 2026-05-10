# Investigation Notes - CA MFA Bypass

## Detection Trigger

During routine review of Entra ID interactive sign-in logs, multiple successful
authentications were observed for the CEO account without MFA enforcement.

The CA policy enforcing MFA was enabled and active for all other users.

## Sign-In Log Observations

| Field | Observed Value | Expected Value |
|---|---|---|
| Conditional Access Status | Not Applied | Success |
| Authentication Requirement | Single-factor | Multifactor |
| Applications | OfficeHome, SharePoint, OWA | Same |
| Location | Toronto, Ontario, Canada | Consistent |
| Sign-in Result | Successful | Successful |

Comparison with earlier sign-in events confirmed MFA had previously been
enforced for this account, ruling out a platform or authentication failure.

## Root Cause Identification

Review of CA configuration revealed the CEO account was explicitly excluded
from the policy enforcing MFA. CA evaluation was bypassed entirely for this
user, allowing single-factor authentication to succeed.

This is a **policy exception abuse** condition, not a platform failure.

## Investigation Scope Assessment

| Area | Finding |
|---|---|
| Other excluded accounts | None identified |
| Malicious sign-in activity during exposure | No indicators found |
| Data access anomalies | None identified |
| Lateral movement | No evidence |

The exposure was limited to authentication downgrade on a single account.
No evidence of exploitation during the exposure window.

## Detection Evidence

| # | Screenshot | Description |
|---|---|---|
| DE-01 | P2-DE-SignIn-Logs-MFA-Bypass-01.png | Sign-in logs showing CA Not Applied, single-factor auth |
| DE-02 | P2-DE-CA-Policy-Explicit-Exclusion-03.png | CA policy exclusion - CEO account confirmed |

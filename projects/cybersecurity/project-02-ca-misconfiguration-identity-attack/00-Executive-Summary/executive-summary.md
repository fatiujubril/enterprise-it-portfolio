# Executive Summary

## Incident Overview

| Field | Detail |
|---|---|
| **Incident Type** | Conditional Access Exception Abuse - MFA Bypass |
| **Attack Vector** | Internal policy misconfiguration |
| **Affected Identity** | CEO - high-value cloud identity |
| **Exposure Window** | Duration of unapproved CA exclusion |
| **Impact** | Single-factor authentication on executive account |
| **Detection Source** | Manual review of Entra ID sign-in logs |
| **Resolution** | Fully contained and remediated |

## What Happened

The CEO experienced an authentication disruption after a mobile device change
invalidated their Microsoft Authenticator registration. To restore access quickly,
IT excluded the CEO account from the Conditional Access policy enforcing MFA.

The exclusion had no expiration date, no approval record, and no monitoring.
As a result, the CEO account silently authenticated using single-factor authentication
while the environment appeared compliant at a policy configuration level.

## How It Was Detected

Routine review of Entra ID interactive sign-in logs revealed that the CEO account
was successfully authenticating with Conditional Access status of Not Applied and
authentication requirement of single-factor authentication.

Cross-referencing with the CA policy configuration confirmed the explicit exclusion
as the root cause.

## What Was Done

1. CEO account removed from CA policy exclusion list
2. Conditional Access enforcement immediately restored
3. CEO re-enrolled Microsoft Authenticator on new mobile device
4. Post-recovery sign-in logs confirmed MFA enforcement applied
5. Governance recommendations documented to prevent recurrence

## Key Findings

- MFA registration alone does not guarantee MFA enforcement
- CA exceptions without expiration or review create silent long-term risk
- High-value identities need stricter exception governance, not looser controls
- Sign-in telemetry is more reliable than policy configuration review alone

## Outcome

Full Conditional Access enforcement restored. No evidence of malicious exploitation
of the exposure window. Governance controls recommended to prevent recurrence.

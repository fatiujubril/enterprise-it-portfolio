# Root Cause Analysis

## Primary Cause

A Conditional Access policy exception was introduced without expiration,
approval tracking, or follow-up review. The exception caused the CA policy
to evaluate as Not Applied for the CEO account, downgrading authentication
to single-factor without any automated alert or detection.

## Root Cause Chain

```
CEO device change invalidates Authenticator registration
      |
      v
MFA challenges cannot be completed - business access disrupted
      |
      v
IT creates CA exception under business pressure [GOVERNANCE GAP]
      |
      v
No expiration, no review, no monitoring configured [CONTROL GAP]
      |
      v
CA evaluates Not Applied - single-factor auth succeeds [EXPOSURE]
      |
      v
No automated alert fires [DETECTION GAP]
      |
      v
Exposure persists until manual telemetry review [DISCOVERY]
```

## Contributing Factors

| Factor | Detail | Impact |
|---|---|---|
| No exception governance process | CA exclusions added ad-hoc without approval | High - exception created without security review |
| No time-bound expiration | Exclusion had no automatic end date | High - indefinite exposure window |
| No alert on CA Not Applied | No monitoring for policy bypass conditions | High - exposure was silent |
| MFA usability not resolved | Root device issue not addressed at time of exception | Medium - exception became permanent workaround |

## Key Insight

The CA policy itself was correctly designed. The failure occurred entirely in
the governance layer around exception handling - not in the technology.

This is the most important finding: **identity security governance failures
are often more dangerous than technical misconfigurations**, because they
are harder to detect, persist longer, and are frequently repeated.

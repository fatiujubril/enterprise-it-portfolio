# Detection Logic - Conditional Access MFA Bypass

## Detection Objective

Identify high-value accounts successfully authenticating without MFA due to
Conditional Access exclusions, despite an active MFA enforcement policy.

## Detection Strategy

Detection is based on **sign-in telemetry analysis** rather than policy
configuration review. Policy existence does not guarantee enforcement.
Authentication outcome fields are the authoritative signal.

## Primary Detection Conditions

Alert when ALL of the following are true:

| Condition | Value |
|---|---|
| Sign-in type | Interactive user sign-in |
| Authentication result | Successful (ResultType == 0) |
| Authentication requirement | Single-factor authentication |
| Conditional Access status | Not Applied |
| User context | Executive, privileged, or business-critical account |

## Detection Signal Sources

| Source | Signal |
|---|---|
| Entra ID Sign-in Logs | Auth requirement + CA evaluation result |
| CA Evaluation Details | Policy name + applied/not applied status |
| Audit Logs | CA policy modification history |

## False Positive Considerations

| Account Type | Handling |
|---|---|
| Break-glass emergency accounts | Exclude from alert scope |
| Service accounts | Exclude if non-interactive |
| Time-bound approved exceptions | Document and tag for review |

## Risk Classification

| Attribute | Value |
|---|---|
| **Severity** | High |
| **Category** | Identity Misconfiguration |
| **Technique** | Authentication Policy Bypass |
| **MITRE** | T1556 - Modify Authentication Process |

## Expected Analyst Response

1. Validate CA policy configuration and exclusion list
2. Confirm whether exclusion is approved and time-bound
3. Escalate to IAM team if unjustified
4. Initiate containment if exposure confirmed
5. Document findings and remediation actions

## Key Insight

**Configuration-level review is insufficient.** A policy can appear fully
enabled while silently not applying to specific accounts. Authentication
outcome telemetry is the only reliable validation method.

# Detection Logic – Conditional Access MFA Bypass

This document outlines the detection logic that would be used by a security operations team to identify Conditional Access misconfigurations resulting in multifactor authentication (MFA) bypass.

The focus is on **policy outcome validation**, rather than policy existence alone.

---

## Detection Objective

Identify high-value user accounts successfully authenticating without MFA due to Conditional Access exclusions or misconfigurations, despite an active MFA enforcement policy.

---

## Detection Strategy

Detection is based on **sign-in telemetry analysis** rather than configuration review alone. The primary indicator of concern is successful interactive authentication events where:

- A Conditional Access policy enforcing MFA exists
- Conditional Access evaluation reports **Not Applied**
- Authentication requirement is downgraded to **single-factor**
- The affected account is a high-value or privileged identity

---

## Primary Detection Conditions

A detection alert should be generated when **all** of the following conditions are met:

1. **Sign-in Type**
   - Interactive user sign-in

2. **Authentication Result**
   - Status = Successful

3. **Authentication Requirement**
   - Single-factor authentication

4. **Conditional Access Evaluation**
   - Conditional Access = Not Applied

5. **User Context**
   - Executive, privileged, or business-critical account
   - OR member of a high-value identity group

---

## Detection Signal Sources

The following Microsoft Entra ID telemetry sources are required:

- Interactive sign-in logs
- Conditional Access evaluation results
- Authentication requirement field
- User and group context metadata

Configuration state alone is insufficient, as Conditional Access exclusions override policy intent without generating errors.

---

## False Positive Considerations

Potential false positives may include:

- Break-glass accounts explicitly designed for emergency access
- Service accounts exempt from interactive authentication policies
- Initial tenant setup or testing accounts in controlled lab environments

To reduce false positives, detection logic should exclude:

- Known emergency access accounts
- Documented and approved exception groups
- Time-bound exclusions with valid approval records

---

## Risk Classification

This condition should be classified as:

- **Severity:** High  
- **Category:** Identity Misconfiguration  
- **Attack Technique:** Authentication Policy Bypass  
- **MITRE ATT&CK Mapping:**  
  - TA0006 – Credential Access  
  - T1556 – Modify Authentication Process (Policy Exception Abuse)

---

## Expected Analyst Action

Upon detection, the analyst should:

1. Validate Conditional Access policy configuration
2. Identify any explicit user or group exclusions
3. Confirm whether the exclusion is approved and time-bound
4. Escalate to identity or IAM administrators if unjustified
5. Initiate containment actions if exposure is confirmed

---

## Outcome

This detection logic ensures that Conditional Access enforcement is continuously validated based on **real authentication outcomes**, not assumed policy coverage, reducing the risk of silent MFA bypass conditions for high-value identities.

# Tenant Overview

## Lab Environment

This project was conducted in a dedicated Microsoft 365 Business tenant created
specifically for identity security testing and Conditional Access validation.

The environment was kept minimal to ensure all observed behaviors and sign-in
events could be clearly attributed to Conditional Access configuration decisions
rather than background platform noise or inherited legacy settings.

## Tenant Characteristics

| Component | Detail |
|---|---|
| **Tenant Type** | Microsoft 365 Business |
| **Identity Provider** | Microsoft Entra ID (cloud-only) |
| **Licensing** | Microsoft 365 Business Standard + Entra ID P2 (trial) |
| **Directory State** | Newly created - no pre-existing policies |
| **User Model** | Native cloud users only (no hybrid identities) |

## Authentication Baseline

At baseline the tenant was configured with modern authentication controls:

- MFA available and properly configured
- Microsoft Authenticator enabled as primary authentication method
- Conditional Access policy requiring MFA for all interactive sign-ins
- No users excluded from Conditional Access enforcement
- Sign-in logs confirmed MFA-protected authentications with no anomalies

## Scope

**In Scope:**
- Microsoft Entra ID Conditional Access
- MFA enforcement and authentication behavior
- Interactive user sign-in telemetry
- Identity misconfiguration and exception handling

**Out of Scope:**
- Endpoint compromise or malware
- Email phishing or credential harvesting
- Azure infrastructure workloads
- Microsoft Sentinel or SIEM correlation

## Baseline Evidence

| # | Screenshot | Description |
|---|---|---|
| 01 | [01-P2-BL-ENTRA-Tenant-Overview.png](../Evidence/screenshots/baseline/01-P2-BL-ENTRA-Tenant-Overview.png) | Entra ID tenant details and licensing at baseline |
| 03 | [03-P2-BL-Auth-Methods-Configuration.png](../Evidence/screenshots/baseline/03-P2-BL-Auth-Methods-Configuration.png) | Authentication methods - Authenticator enabled |
| 04 | [04-P2-BL-SignIn-Logs-Clean.png](../Evidence/screenshots/baseline/04-P2-BL-SignIn-Logs-Clean.png) | Baseline sign-in logs - MFA enforced, no anomalies |

# Environment Overview

## Lab Configuration

| Component | Detail |
|---|---|
| **Identity Platform** | Microsoft Entra ID |
| **Email Platform** | Exchange Online |
| **Endpoint** | Windows 11 (domain-joined) |
| **User Privilege** | Standard (non-administrative) |
| **MFA Status** | Registered - not enforced at time of compromise |
| **Conditional Access** | Not applied to Outlook Web at baseline |

## Tenant Setup

- Microsoft 365 Business Premium lab tenant
- Single test user configured as a standard domain user
- No local or domain administrative privileges assigned
- Exchange Online mailbox provisioned with default settings
- Entra ID sign-in logs and Exchange audit logs enabled

## Security Controls at Baseline

| Control | Status |
|---|---|
| MFA | Registered, not enforced |
| Conditional Access | Not configured |
| Inbox Rule Alerting | Not configured |
| Legacy Auth Block | Not configured |
| Sign-in Risk Policy | Not configured |

This baseline reflects a common real-world state in organizations that have partially
deployed Microsoft 365 security features but have not yet enforced them via policy.

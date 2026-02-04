## Detection – Conditional Access MFA Bypass

During review of Microsoft Entra ID interactive sign-in logs, multiple successful authentications were observed for the CEO account without multifactor authentication enforcement.

Sign-in telemetry showed:
- Conditional Access: Not Applied
- Authentication requirement: Single-factor authentication
- Affected applications: OfficeHome, SharePoint Online, Outlook Web
- Source location: Toronto, Ontario, CA

Comparison with earlier sign-in events confirmed that MFA enforcement was previously successful for the same user, indicating a configuration-based exclusion rather than a platform or authentication failure.

### Evidence

![CEO MFA Bypass Sign-In Logs](../evidence/screenshots/detection/P2-DE-SignIn-Logs-MFA-Bypass-01.png)

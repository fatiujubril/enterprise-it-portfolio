# Detection – Conditional Access MFA Bypass

This section documents how the Conditional Access misconfiguration was identified through review of Microsoft Entra ID sign-in telemetry.

---

## Detection Summary

During routine review of Microsoft Entra ID interactive sign-in logs, multiple successful authentications were observed for the Chief Executive Officer (CEO) account without multifactor authentication (MFA) enforcement.

Although a Conditional Access policy requiring MFA was enabled in the tenant, authentication telemetry indicated that the policy was **not applied** to the affected user.

---

## Sign-In Log Observations

Analysis of interactive sign-in events revealed the following conditions:

- **Conditional Access:** Not Applied  
- **Authentication requirement:** Single-factor authentication  
- **Affected applications:** OfficeHome, SharePoint Online, Outlook Web  
- **Source location:** Toronto, Ontario, Canada  
- **Sign-in result:** Successful  

Comparison with earlier sign-in events confirmed that MFA enforcement had previously been successful for the same account, indicating that the behavior change was not caused by a platform or authentication failure.

---

## Root Cause Identification

Further review of Conditional Access configuration revealed that the CEO account was explicitly excluded from the policy enforcing MFA. As a result, Conditional Access evaluation was bypassed entirely for this user, allowing single-factor authentication to succeed.

This condition represents a **policy exception abuse**, not a failure of the Conditional Access platform itself.

---

## Detection Evidence

### Evidence 4.1 – MFA Bypass Observed in Sign-In Logs

This evidence shows successful interactive sign-ins for the CEO account where Conditional Access was not applied and MFA was not required.

![Evidence 4.1 – MFA Bypass Sign-In Logs](../evidence/screenshots/detection/P2-DE-SignIn-Logs-MFA-Bypass-01.png)

---

### Evidence 4.2 – Conditional Access Policy Exclusion (Root Cause)

This evidence confirms that the CEO account was explicitly excluded from the Conditional Access policy enforcing MFA, explaining why MFA was not applied during sign-in.

![Evidence 4.2 – Conditional Access Policy Exclusion](../evidence/screenshots/detection/P2-DE-CA-Policy-Explicit-Exclusion-03.png)

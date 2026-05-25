# Lessons Learned
## FatiuLab Security — Post-Tabletop Exercise Review

**Exercise Reference:** INC-2026-001 Tabletop  
**Exercise Date:** May 2026  
**Facilitator:** Global Administrator  
**Participants:** Global Administrator, Security Administrator  
**Document Owner:** Global Administrator  
**Last Updated:** May 2026

---

## 1. Executive Summary

The tabletop exercise simulating a compromised Security Administrator account and Azure privilege abuse revealed that the current detection and response capabilities are effective for the mid-to-late stages of an identity attack but have meaningful gaps in early detection and response speed.

**Key findings:**

- Detection rules fired correctly for 4 of 6 attack events
- The initial AiTM credential theft was not detected until privilege escalation began
- Attacker dwell time before first detection was approximately 90 minutes
- Response actions were clear and well-defined once the incident was declared
- The PIM approval gate successfully blocked Global Administrator escalation
- Storage network isolation successfully prevented data exfiltration

**Overall assessment:** The program provides a solid defensive foundation. The primary gap is early detection of the initial compromise — currently the program detects what attackers do with access, not the act of gaining access itself.

---

## 2. What Worked Well

### 2.1 PIM Approval Gate Held
The requirement for approval to activate Global Administrator was the single most effective control in this scenario. The attacker was unable to escalate to Global Admin despite having Security Administrator access and attempting activation twice.

**Lesson:** Approval gates for highest-privilege roles are a critical chokepoint. Do not remove or weaken the GA approval requirement.

### 2.2 Storage Network Isolation Prevented Exfiltration
The attacker's primary objective — accessing payment data in fatiulabstorage01 — was blocked by the public network access disabled setting. The attacker could not reach the storage account despite having Owner access on the subscription.

**Lesson:** Defense in depth works. Even with subscription-level Owner access, a compromised attacker cannot access properly isolated resources. Network isolation is a critical compensating control.

### 2.3 Detection Rules Correlated Correctly
DET-01, DET-04, DET-06, and DET-08 all fired during the attack chain and were correctly correlated by Sentinel into a coherent incident. The combination of alerts told the full story — PIM activation, CA policy change, Azure role assignment, resource deployment.

**Lesson:** Detection rule correlation is working as designed. The combination of rules provides a richer picture than any individual rule alone.

### 2.4 CA Policy Re-enablement Was Fast
CA03 was re-enabled within 3 minutes of containment starting. The response team knew exactly what to fix and where to fix it.

**Lesson:** Documented policy configurations pay off in incidents. Knowing the expected state made restoration fast and confident.

---

## 3. What Did Not Work Well

### 3.1 90-Minute Dwell Time Before Detection
The attacker had 90 minutes of undetected access from initial compromise (9:15 PM) to first Sentinel alert (10:45 PM). During this time they disabled a CA policy, assigned Owner role in Azure, and deployed infrastructure.

**Root cause:** Scheduled detection rules run hourly. The AiTM compromise itself generated no immediate alert. Detection only started when the attacker took privileged actions that triggered rules.

**Impact:** 90 minutes is a significant window in a FinTech environment. An attacker with more time could have caused greater damage.

### 3.2 No Detection for AiTM Session Hijack
The initial credential theft via adversary-in-the-middle proxy bypassed MFA entirely. There was no detection rule covering this specific attack vector. DET-05 (Impossible Travel) would only fire if the attacker signed in from a foreign country — a proxy with domestic exit nodes would evade this completely.

**Root cause:** Current MFA implementation uses push notification / TOTP — both are susceptible to AiTM proxy attacks. Phishing-resistant MFA (FIDO2/passkeys) would prevent this attack vector entirely.

### 3.3 No Alert for After-Hours Legacy Auth Re-enablement
CA03 was disabled at 9:52 PM on a Monday night. While DET-04 fired for the CA policy change, there was no additional context in the alert about the severity of this specific change — disabling the legacy auth block is significantly more dangerous than updating a policy description.

**Root cause:** DET-04 treats all CA policy changes equally. No severity differentiation based on which policy was changed or what was changed.

### 3.4 No Automated Containment
All containment actions required manual intervention by the Global Admin. The time between first alert (10:45 PM) and account disabled (10:57 PM) was 12 minutes — during which the attacker submitted a GA PIM request.

**Root cause:** No automation playbooks configured in Sentinel. Manual response is slower and requires the Global Admin to be available and responsive at 10:45 PM on a Monday night.

---

## 4. Improvement Backlog

The following improvements are prioritized based on impact on the identified gaps:

---

### IMP-01 — Deploy Phishing-Resistant MFA (FIDO2)
**Priority:** Critical  
**Gap addressed:** AiTM credential theft bypassing push notification MFA  
**Description:** Replace Microsoft Authenticator push notifications with FIDO2 security keys or Windows Hello for Business for all privileged accounts. Phishing-resistant MFA cannot be captured by AiTM proxies because the authentication is bound to the origin domain.  
**Effort:** High — requires hardware keys or compatible devices  
**Timeline:** 30 days

---

### IMP-02 — Sentinel Automation Playbook for Compromised Account
**Priority:** High  
**Gap addressed:** 12-minute manual response time for account containment  
**Description:** Create a Sentinel automation playbook triggered by correlated high-severity identity alerts that automatically:
1. Disables the flagged account
2. Revokes all active sessions
3. Sends immediate notification to Global Admin and Security Admin
4. Creates a P1 incident with pre-populated containment checklist

**Effort:** Medium — Logic Apps integration with Microsoft Graph API  
**Timeline:** 14 days

---

### IMP-03 — Enhanced DET-04 — CA Policy Severity Tiering
**Priority:** High  
**Gap addressed:** No differentiation between high-risk and low-risk CA policy changes  
**Description:** Update DET-04 to assign different severities based on which policy was changed:
- CA03 disabled → Critical severity
- CA01/CA02 disabled → Critical severity
- Any policy deleted → High severity
- Policy updated → Medium severity

```kql
| extend AlertSeverity = case(
    PolicyName contains "Legacy" and OperationName == "Delete conditional access policy", "Critical",
    PolicyName contains "MFA" and OperationName == "Delete conditional access policy", "Critical",
    OperationName == "Delete conditional access policy", "High",
    "Medium"
)
```
**Effort:** Low — KQL update only  
**Timeline:** 7 days

---

### IMP-04 — Add Detection for Sign-In Token Anomalies
**Priority:** High  
**Gap addressed:** AiTM session hijack not detected  
**Description:** Add a new detection rule (DET-09) that identifies anomalous session characteristics associated with AiTM attacks — sign-ins where the user agent, IP, and session token properties suggest proxy usage.

```kql
SigninLogs
| where ResultType == 0
| where UserPrincipalName in (admin_accounts)
| extend IsAnomalous = RiskLevelDuringSignIn in ("high", "medium")
    or RiskEventTypes_V2 has "anonymizedIPAddress"
    or RiskEventTypes_V2 has "unfamiliarFeatures"
| where IsAnomalous == true
```
**Effort:** Medium — requires Identity Protection P2 integration  
**Timeline:** 21 days

---

### IMP-05 — Convert High-Priority Detection Rules to NRT
**Priority:** Medium  
**Gap addressed:** 90-minute dwell time from scheduled rules  
**Description:** Convert DET-02, DET-04, and DET-06 from Scheduled (hourly) to NRT rules. These three rules cover the most critical identity attack actions and should generate alerts within 1-2 minutes rather than up to 60 minutes.  
**Effort:** Low — rule type change only  
**Timeline:** 7 days

---

### IMP-06 — PIM Alert Configuration
**Priority:** Medium  
**Gap addressed:** No proactive alerting on suspicious PIM patterns  
**Description:** Enable built-in PIM alerts in Entra ID for:
- Roles are being activated too frequently
- Roles don't require MFA during activation
- Roles have duplicate assignments
- There are too many Global Administrators

**Effort:** Low — configuration only  
**Timeline:** 7 days

---

### IMP-07 — Privileged Access Workstation (PAW) Policy
**Priority:** Medium  
**Gap addressed:** Admin accounts accessible from standard workstations  
**Description:** Implement a Named Location or device compliance CA policy requiring admin accounts to sign in only from compliant devices. This prevents an attacker from using stolen credentials from an unmanaged device — even if they have the password and MFA token.  
**Effort:** Medium — requires Intune device management  
**Timeline:** 30 days

---

## 5. Improvement Summary

| ID | Improvement | Priority | Effort | Timeline |
|---|---|---|---|---|
| IMP-01 | Phishing-resistant MFA (FIDO2) | Critical | High | 30 days |
| IMP-02 | Sentinel automation playbook | High | Medium | 14 days |
| IMP-03 | DET-04 severity tiering | High | Low | 7 days |
| IMP-04 | AiTM session anomaly detection | High | Medium | 21 days |
| IMP-05 | Convert key rules to NRT | Medium | Low | 7 days |
| IMP-06 | PIM alert configuration | Medium | Low | 7 days |
| IMP-07 | PAW policy | Medium | Medium | 30 days |

---

## 6. Updated Risk Register Items

The following risks in the risk register should be updated based on tabletop findings:

| Risk ID | Risk | Update |
|---|---|---|
| RISK-01 | Compromised Global Admin | Likelihood remains 3 — AiTM bypass of MFA confirmed feasible |
| RISK-05 | MFA Authentication Bypass | Likelihood upgraded to 3 — AiTM is a realistic and widely used technique |
| RISK-11 | Insider Threat | No change — existing controls effective |

---

## 7. Exercise Conclusion

The tabletop exercise was successful in achieving its objectives:

✅ Detection rules validated — 4 of 6 attack events detected  
✅ Response process tested — containment actions clear and fast  
✅ Gaps identified — dwell time, AiTM detection, automation  
✅ Improvement backlog created — 7 concrete, actionable items  
✅ Risk register updated — 2 risks updated based on findings  

The program provides a strong defensive foundation with clear improvement priorities. The most impactful next steps are phishing-resistant MFA (IMP-01) and automated containment playbooks (IMP-02) — together these would reduce attacker dwell time from 90 minutes to near-zero for the scenario tested.

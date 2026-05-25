# Incident Timeline
## FatiuLab Security — Compromised Admin + Azure Privilege Abuse

**Incident Reference:** INC-2026-001  
**Classification:** P1 — Critical  
**Status:** Tabletop Exercise  
**Exercise Date:** May 2026

---

## 1. Attack Timeline

| Time | Phase | Event | Actor | Detection |
|---|---|---|---|---|
| 9:00 PM | Reconnaissance | Attacker sends spear phishing email to Security Admin | Attacker | None |
| 9:15 PM | Initial Access | Security Admin clicks phishing link — credentials captured via AiTM proxy | Attacker | None |
| 9:15 PM | Initial Access | Attacker obtains authenticated session cookie — MFA bypassed | Attacker | DET-05 (Impossible Travel) — fires if sign-in location differs |
| 9:47 PM | Privilege Escalation | Attacker activates Security Administrator via PIM — justification entered | Attacker | **DET-01 fires** — outside business hours |
| 9:52 PM | Defense Evasion | Attacker disables CA03-Block-Legacy-Authentication | Attacker | **DET-04 fires** — CA policy modified |
| 9:55 PM | Privilege Escalation | Attacker attempts Global Administrator PIM activation — blocked by approval | Attacker | Approval request queued — not auto-detected |
| 10:15 PM | Privilege Escalation | Attacker assigns Owner role on Azure subscription 1 directly | Attacker | **DET-06 fires** — Azure role assignment |
| 10:30 PM | Persistence | Attacker deploys VM and storage account in new resource group | Attacker | **DET-08 fires** — Defender high severity |
| 10:35 PM | Collection | Attacker attempts access to fatiulabstorage01 — blocked by network controls | Attacker | None — blocked before access |
| 10:38 PM | Persistence | Attacker attempts legacy auth sign-in — fails (no legacy password set) | Attacker | None — failed attempt |
| 10:45 PM | Privilege Escalation | Attacker submits GA PIM activation request | Attacker | Approval queue alert |

---

## 2. Response Timeline

| Time | Phase | Action | Actor | Outcome |
|---|---|---|---|---|
| 10:45 PM | Detection | Sentinel incident created — DET-01 PIM outside hours | Sentinel | Alert generated |
| 10:46 PM | Detection | Sentinel incident created — DET-04 CA policy modified | Sentinel | Alert generated |
| 10:46 PM | Detection | Sentinel incident created — DET-06 Azure role assignment | Sentinel | Alert generated |
| 10:47 PM | Detection | Defender for Cloud email notification — high severity findings | Defender | Email to security contact |
| 10:50 PM | Triage | On-call Security Admin reviews Sentinel dashboard — multiple correlated alerts | Security Admin | Incident declared P1 |
| 10:52 PM | Notification | Global Admin notified — P1 incident, Security Admin account suspected compromised | Security Admin | Response team assembled |
| 10:55 PM | Containment | Global Admin re-enables CA03-Block-Legacy-Authentication | Global Admin | Legacy auth re-blocked |
| 10:57 PM | Containment | Global Admin disables Security Admin account in Entra ID | Global Admin | Account disabled |
| 10:57 PM | Containment | Global Admin revokes all active sessions for Security Admin account | Global Admin | All sessions terminated |
| 10:58 PM | Containment | Global Admin deactivates Security Admin PIM role activation | Global Admin | Security Admin role removed |
| 11:00 PM | Containment | Global Admin denies GA PIM activation request | Global Admin | Escalation blocked |
| 11:05 PM | Containment | Global Admin removes Owner role assignment from Security Admin account | Global Admin | Azure privilege removed |
| 11:10 PM | Investigation | Global Admin reviews Sentinel logs — establishes full attack timeline | Global Admin | Timeline confirmed |
| 11:20 PM | Eradication | Attacker-deployed resources deleted — VM, storage, resource group | Global Admin | Attacker infrastructure removed |
| 11:30 PM | Recovery | CA policy set reviewed — all 5 policies confirmed in correct state | Global Admin | Policy set restored |
| 11:35 PM | Recovery | New Security Admin account created — separate from compromised account | Global Admin | Access restored for legitimate admin |
| 11:45 PM | Recovery | MFA reset for new Security Admin — Microsoft Authenticator re-enrolled | Security Admin | MFA restored |
| 12:00 AM | Post-Incident | Incident documented — lessons learned initiated | Global Admin | INC-2026-001 created |

---

## 3. Detection Coverage Analysis

| Attack Event | Detected | Detection Method | Time to Detect |
|---|---|---|---|
| AiTM credential theft | Partial | DET-05 if sign-in from different country | ~1 hour (scheduled) |
| PIM activation outside hours | ✅ Yes | DET-01 | ~1 hour (scheduled) |
| CA policy disabled | ✅ Yes | DET-04 | ~1 hour (scheduled) |
| GA PIM activation attempt | Partial | Approval queue — no auto-alert | Manual |
| Azure Owner role assignment | ✅ Yes | DET-06 | ~1 hour (scheduled) |
| VM deployment | ✅ Yes | DET-08 | ~1 hour (scheduled) |
| Storage access attempt | N/A | Blocked by network controls | N/A |
| Legacy auth attempt | No | No detection rule for failed legacy auth | Not detected |

**Detection rate: 4 of 6 attack events detected (67%)**

**Key gap identified:** The initial AiTM credential theft and session hijack generated no immediate alert. DET-05 (Impossible Travel) would only fire if the attacker's IP was in a different country — if the attacker proxied through a domestic IP, this detection would not fire.

---

## 4. Dwell Time Analysis

| Metric | Value |
|---|---|
| Time of initial compromise | 9:15 PM |
| Time of first detection | 10:45 PM (DET-01) |
| **Attacker dwell time before detection** | **~90 minutes** |
| Time of containment (account disabled) | 10:57 PM |
| Time of full eradication | 11:20 PM |
| Total incident duration | ~2 hours 5 minutes |

**90-minute dwell time before detection** is the primary finding of this exercise. The attacker had 90 minutes of undetected access during which they:
- Disabled a CA policy
- Assigned Owner role in Azure
- Deployed malicious infrastructure
- Attempted data access

Reducing dwell time is the primary improvement goal from this exercise.

---

## 5. Control Effectiveness Summary

| Control | Effective? | Notes |
|---|---|---|
| CA01 MFA all users | ⚠️ Partial | MFA enforced but bypassed via AiTM — phishing-resistant MFA needed |
| CA02 MFA admins | ⚠️ Partial | Same — AiTM bypasses push notification MFA |
| CA03 Block legacy auth | ✅ Yes | Initially bypassed but quickly re-enabled — legacy auth attempt failed |
| CA04 MFA Azure Management | ✅ Yes | Azure Management MFA not bypassed — attacker used existing session |
| PIM GA approval | ✅ Yes | GA activation blocked — approval gate held |
| PIM 4-hour window | ✅ Yes | Security Admin window would have expired |
| Storage network isolation | ✅ Yes | Data exfiltration from fatiulabstorage01 blocked |
| DET-01 PIM Outside Hours | ✅ Yes | Fired correctly |
| DET-04 CA Policy Modified | ✅ Yes | Fired correctly |
| DET-06 Azure Role Assignment | ✅ Yes | Fired correctly |
| DET-08 Defender Finding | ✅ Yes | Fired on VM deployment |

# Incident Tabletop — Scenario
## FatiuLab Security — Compromised Admin + Azure Privilege Abuse

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**Exercise Type:** Tabletop Exercise  
**Scenario Classification:** Identity Compromise + Cloud Privilege Abuse  
**Exercise Date:** May 2026  
**Facilitator:** Global Administrator  
**Participants:** Global Administrator, Security Administrator

---

## 1. Exercise Overview

This tabletop exercise simulates a realistic attack scenario against FatiuLab Security's identity and cloud environment. The purpose is to validate the detection and response capabilities built throughout this program, identify gaps in the response process, and produce actionable improvements.

**Exercise objectives:**

1. Validate that detection rules trigger at the right points in the attack chain
2. Test the incident response process — who does what, and in what order
3. Identify gaps between detection capability and response readiness
4. Produce a lessons learned backlog for program improvement

**Exercise ground rules:**

- This is a discussion-based exercise — no actual systems are touched
- All participants respond as if events are happening in real time
- The facilitator presents injects; participants respond with their actions
- There are no wrong answers — the goal is learning, not testing

---

## 2. Scenario Background

**Victim organization:** FatiuLab Security — 200-person FinTech company processing customer payment data

**Threat actor:** External attacker — financially motivated, likely affiliated with a cybercriminal group targeting FinTech companies

**Attack objective:** Gain persistent access to FatiuLab Security's Azure environment to exfiltrate payment data and deploy crypto mining infrastructure using the company's Azure credits

**Initial access vector:** Spear phishing email targeting the Security Administrator

---

## 3. Scenario Narrative

### Phase 1 — Initial Compromise

**Date:** Monday, 9:15 PM

A Security Administrator at FatiuLab Security receives a convincing phishing email appearing to come from Microsoft, warning of suspicious activity on their account. The email contains a link to a fake Microsoft sign-in page that captures credentials and MFA tokens via an adversary-in-the-middle (AiTM) proxy.

The Security Administrator enters their credentials and completes the MFA challenge on the fake page. The attacker's proxy relays the session cookie to a server in Eastern Europe, granting the attacker an authenticated session as the Security Administrator — bypassing MFA entirely despite it being enforced.

**What the attacker has at this point:**
- Authenticated session as securityadmin@FatiuLabSecurity.onmicrosoft.com
- Access to all Microsoft 365 applications
- No privileged Azure or Entra ID role yet — Security Administrator is eligible only via PIM

---

### Phase 2 — Privilege Escalation

**Date:** Monday, 9:47 PM

Using the compromised Security Administrator session, the attacker navigates to PIM and activates the Security Administrator role. They enter a plausible-sounding justification: *"Investigating potential security incident."*

The attacker now has Security Administrator privileges for 4 hours. They use this access to:
1. Navigate to Conditional Access policies
2. Disable CA03 (Block Legacy Authentication) — re-enabling legacy auth protocols
3. Attempt to activate Global Administrator — blocked by approval requirement

Frustrated by the GA approval gate, the attacker takes a different path. Using Security Administrator access, they navigate to Azure and assign themselves the Owner role on Azure subscription 1 directly — bypassing PIM entirely since Azure RBAC assignments don't go through Entra PIM in this configuration.

**What the attacker has at this point:**
- Security Administrator role (4 hour window)
- Owner access on Azure subscription 1 (permanent, direct assignment)
- Legacy authentication re-enabled — new attack vector available

---

### Phase 3 — Persistence and Impact

**Date:** Monday, 10:30 PM

With Owner access on the Azure subscription, the attacker:
1. Creates a new resource group: `RG-Attacker-Persistence`
2. Deploys a virtual machine for crypto mining
3. Creates a new storage account with public access enabled
4. Attempts to exfiltrate data from fatiulabstorage01 — blocked by network access controls

The attacker attempts to sign in using legacy authentication protocols now that CA03 is disabled. This fails because the account doesn't have a legacy-compatible password set.

The attacker also attempts to use the compromised Security Admin account to access PIM and activate Global Administrator again — the approval request goes to the Global Admin account which is currently unmonitored.

---

### Phase 4 — Detection and Response

**Date:** Monday, 10:45 PM

Sentinel begins generating alerts:

- **DET-01** fires — PIM activation outside business hours (9:47 PM activation)
- **DET-04** fires — CA policy modified (CA03 disabled at 9:52 PM)
- **DET-06** fires — Azure Owner role assigned (10:15 PM)
- **DET-08** fires — Defender for Cloud high severity finding (new VM deployment)

The Global Admin receives email notifications from Defender for Cloud about high severity findings. The on-call Security Administrator (a different person) sees Sentinel incidents in the dashboard.

---

## 4. Exercise Injects

The following injects are presented to participants during the exercise. Participants discuss their response to each inject before the facilitator reveals the next one.

---

**Inject 1 — 9:47 PM**
> "Sentinel has generated an alert: DET-01 — PIM role activation outside business hours. Security Administrator account activated Security Administrator role at 9:47 PM with justification 'Investigating potential security incident.' What do you do?"

**Discussion questions:**
- Is this alert high fidelity or could it be a false positive?
- Who do you contact first?
- Do you disable the account or wait to gather more information?
- What additional context do you look for in the logs?

---

**Inject 2 — 9:52 PM (5 minutes later)**
> "A second Sentinel alert fires: DET-04 — CA Policy Modified. CA03-Block-Legacy-Authentication was disabled at 9:52 PM by the Security Administrator account. What does this change in your assessment?"

**Discussion questions:**
- Does this change the alert from possible false positive to confirmed incident?
- What is the immediate remediation for CA03 being disabled?
- Do you now disable the Security Administrator account?
- Who needs to be notified at this point?

---

**Inject 3 — 10:15 PM**
> "A third alert fires: DET-06 — Azure Privileged Role Assignment. The Security Administrator account assigned Owner role on Azure subscription 1 to itself at 10:15 PM. What do you do?"

**Discussion questions:**
- What is the blast radius of an Owner role assignment?
- What Azure actions do you take immediately?
- How do you remove the Owner role without disrupting legitimate operations?
- Is this now a P1 incident?

---

**Inject 4 — 10:30 PM**
> "Defender for Cloud alerts are firing — a new resource group and virtual machine have been deployed in your subscription. Your Azure cost alert shows unexpected spend beginning. What do you do?"

**Discussion questions:**
- Do you delete the resources immediately or preserve them for forensics?
- Who has the authority to delete production Azure resources?
- What is the order of operations — contain first or investigate first?
- How do you prevent further resource deployment?

---

**Inject 5 — 10:45 PM**
> "The Security Administrator account has submitted a PIM activation request for Global Administrator. The approval request is sitting in the queue. What do you do?"

**Discussion questions:**
- Do you approve or deny the request?
- How do you disable the Security Administrator account?
- Does the Global Admin account need to be protected?
- What actions does the break-glass account enable that normal accounts don't?

---

## 5. Scenario MITRE ATT&CK Mapping

| Phase | Technique | Sub-technique | Detection Rule |
|---|---|---|---|
| Initial Access — AiTM phishing | T1078 | T1078.004 | DET-05 (Impossible Travel) |
| Privilege Escalation — PIM activation | T1078 | T1078.004 | DET-01 (Outside Hours) |
| Defense Evasion — CA policy disabled | T1556 | T1556.009 | DET-04 (CA Modified) |
| Privilege Escalation — Azure Owner | T1098 | T1098.003 | DET-06 (Role Assignment) |
| Persistence — VM deployment | T1578 | T1578.002 | DET-08 (Defender Finding) |
| Privilege Escalation — GA request | T1078 | T1078.004 | DET-02 (GA Outside PIM) |

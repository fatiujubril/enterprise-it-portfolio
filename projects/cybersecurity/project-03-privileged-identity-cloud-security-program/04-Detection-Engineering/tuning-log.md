# Detection Tuning Log
## FatiuLab Security — Analytics Rule False Positive & Tuning Register

**Organization:** FatiuLab Security (FinTech — Payment Processing & Customer Financial Data)  
**SIEM:** Microsoft Sentinel — workspace: law-fatiulab-security  
**Last Updated:** May 2026

---

## 1. Overview

This document records all tuning decisions made to Sentinel analytics rules — including false positive analysis, query modifications, threshold adjustments, and suppression decisions.

Detection engineering is an iterative process. Every rule starts as a hypothesis and improves through operational feedback. This log documents that improvement process, providing:

- An audit trail of every tuning decision
- Rationale for why rules were modified
- Evidence that false positive management is active and deliberate
- A record of what was tested and what the results were

**Tuning philosophy:**

> A rule with zero alerts is not necessarily a good rule — it may have too many suppressions. A rule with too many alerts is not necessarily a bad rule — it may need better scoping. The goal is high-fidelity detection: alert on real attacker behavior, not on noise.

---

## 2. Tuning Log

---

### DET-01 — PIM Activation Outside Business Hours

**Rule deployed:** May 2026  
**Current version:** v1.0  
**Status:** Active — monitoring

**Initial deployment notes:**

Rule deployed with business hours defined as 7am-7pm UTC. This is a conservative definition — actual business hours for a Toronto-based company would be approximately 1pm-11pm UTC (9am-7pm EST). However UTC was chosen for the initial deployment to:

1. Avoid assumptions about timezone — some admins may work remotely from different timezones
2. Cast a wider net initially — better to tune down noise than miss real detections
3. Simplify the query — timezone conversion adds complexity

**Test validation:**

Query tested against 7-day historical data with business hours filter removed. PIM activation events confirmed present in AuditLogs:
- Operation: "Add member to role in PIM completed (timebound)"
- Initiated by: securityadmin@FatiuLabSecurity.onmicrosoft.com
- Activation time: During business hours — correctly excluded by filter

**Planned tuning (v1.1):**

If alert fatigue develops from legitimate after-hours on-call work:
- Add timezone offset: `| extend LocalHour = (ActivationHour + 19) % 24` (UTC to EST)
- Adjust business hours filter to: `LocalHour < 9 or LocalHour > 19`
- Add exclusion for designated on-call accounts

**False positives observed:** None to date

---

### DET-02 — Global Admin Assigned Outside PIM

**Rule deployed:** May 2026  
**Current version:** v1.0  
**Status:** Active — monitoring

**Initial deployment notes:**

High severity rule — any trigger warrants immediate investigation. Rule was tested by verifying the break-glass account GA assignment (15-day duration) would trigger this rule.

**Known exception — break-glass account:**

The break-glass account requires periodic GA assignment renewal (every 15 days due to PIM settings). This will trigger DET-02 when renewed. Decision:

- **Do not suppress** — each renewal should be acknowledged as a conscious decision
- **Document as known exception** — reviewer confirms "this is the break-glass renewal"
- **Rationale:** Suppressing this would mean not detecting if someone else makes an unexpected direct GA assignment at the same time

**Test validation:**

Query confirmed finding "Add member to role" operations in AuditLogs. Break-glass account GA assignment visible as expected trigger scenario.

**False positives observed:** Break-glass renewal — expected, not suppressed

---

### DET-03 — MFA Disabled for Privileged Account

**Rule deployed:** May 2026  
**Current version:** v1.0  
**Status:** Active — monitoring

**Initial deployment notes:**

Rule uses multiple operation names to catch different MFA removal paths. The filter `where InitiatedByUser != TargetUser` was added to exclude self-service MFA updates (user removing their own old device) and focus on admin-initiated changes.

**Potential noise source identified:**

"Update user" is a broad operation name that covers many types of user updates, not just MFA changes. This may generate false positives for other user attribute updates.

**Planned tuning (v1.1):**

Add ModifiedProperty filter to ensure only MFA-related property changes trigger:
```kql
| where ModifiedProperty contains "StrongAuthentication" 
    or ModifiedProperty contains "MFA"
    or OperationName != "Update user"
```

This narrows the "Update user" trigger to only MFA-related property modifications.

**False positives observed:** None to date — monitoring for "Update user" noise

---

### DET-04 — CA Policy Modified or Disabled

**Rule deployed:** May 2026  
**Current version:** v1.0  
**Status:** Active — monitoring

**Initial deployment notes:**

This rule intentionally generates alerts for ALL CA policy changes — including legitimate ones made by authorized admins. The rationale: every CA policy change should be acknowledged and reviewed, not silently pass. This is a governance control as much as a threat detection.

**Lab validation:**

During simulation activities CA05 was toggled Off then back to Report-only. This generated "Update conditional access policy" events in AuditLogs — confirming the rule would have fired.

**High alert volume expected in active environments:**

In a production environment with multiple admins making regular CA policy changes, this rule may generate significant alert volume. Recommended approach for production:

1. Create a change management process — all CA changes require a ticket
2. Add ticket number to alert triage — if ticket exists, close as expected change
3. Alert on changes outside change windows as higher severity

**False positives observed:** CA05 simulation toggle — expected, documented

---

### DET-05 — Impossible Travel on Admin Account

**Rule deployed:** May 2026  
**Current version:** v1.0  
**Status:** Active — monitoring

**Initial deployment notes:**

The 120-minute travel window was chosen as the initial threshold. This is conservative — two hours is physically impossible between most country pairs, but VPN usage will likely be the primary false positive source.

**VPN false positive analysis:**

VPN exits in different countries will trigger this rule even when the user is physically in one location. For example:
- User in Toronto connects to corporate VPN with exit node in the US
- Also signs in directly from Toronto IP
- Rule fires — two different countries within 2 hours

**Planned tuning (v1.1):**

Maintain a VPN exit node IP list as a Sentinel watchlist and exclude those IPs:
```kql
| join kind=leftanti (
    _GetWatchlist('VPN-Exit-Nodes')
    | project SearchKey
) on $left.IPAddress == $right.SearchKey
```

This approach is cleaner than IP range exclusions and is maintainable as VPN ranges change.

**False positives observed:** None to date — VPN tuning planned proactively

---

### DET-06 — Azure Privileged Role Assignment

**Rule deployed:** May 2026  
**Current version:** v1.0  
**Status:** Active — monitoring

**Initial deployment notes:**

Lab validation confirmed the rule detects role assignments. During simulation, a Reader role was assigned to Standard User — this triggered an AzureActivity event. Note: Reader role does not meet the Owner/Contributor filter, confirming the query correctly scopes to high-privilege roles only.

**Known noise source:**

Azure Policy remediation tasks and Defender for Cloud agentless scanning may create service principal role assignments that trigger this rule. These are system-initiated, not human-initiated.

**Planned tuning (v1.1):**

Add Caller exclusion for known system service principals:
```kql
| where Caller !contains "Microsoft.Security"
| where Caller !contains "Microsoft.PolicyInsights"
```

**False positives observed:** None to date — system SP exclusion planned proactively

---

### DET-07 — Break-Glass Account Sign-In

**Rule deployed:** May 2026  
**Current version:** v1.0 — NRT  
**Status:** Active — monitoring

**Initial deployment notes:**

NRT rule — lowest possible detection latency (~1-2 minutes). Lab validation confirmed:
- Break-glass sign-ins visible in SigninLogs
- Multiple sign-in events captured during simulation activity
- Both successful (ResultType == 0) and failed attempts captured

**No suppression policy:**

This rule will never be suppressed or tuned down. Every trigger is a high-priority investigation regardless of frequency. If break-glass is being used legitimately for an emergency, the investigation will confirm that within minutes.

**Automation playbook — planned:**

Future enhancement: attach an automation playbook to this rule that:
1. Creates a high-priority incident automatically
2. Sends immediate email notification to Security Admin and Global Admin
3. Logs the event to the GRC evidence register

**False positives observed:**

Simulation activity on May 24, 2026 — break-glass sign-in for detection testing. Documented as expected test activity. Rule confirmed working.

---

### DET-08 — Defender for Cloud High Severity Finding

**Rule deployed:** May 2026  
**Current version:** v1.0  
**Status:** Active — monitoring

**Initial deployment notes:**

Rule queries AzureActivity for Defender for Cloud security events with High or Critical severity. Initial testing showed the CategoryValue and ActivityStatusValue filters may need adjustment as Defender for Cloud event schema can vary.

**Schema validation needed:**

The Properties field parsing (`parse_json(Properties).severity`) may return empty if the Defender for Cloud event format differs from expected. Recommend validating with actual high-severity Defender events before relying on this rule.

**Alternative approach if schema issues arise:**

Use SecurityAlert table directly if available:
```kql
SecurityAlert
| where AlertSeverity in ("High", "Critical")
| where ProductName contains "Azure Defender"
    or ProductName contains "Microsoft Defender for Cloud"
| project TimeGenerated, AlertName, AlertSeverity, CompromisedEntity, Description
```

**Planned tuning (v1.1):**

After first real Defender alert fires — validate Properties field parsing and adjust if needed.

**False positives observed:** None to date — schema validation pending first real event

---

## 3. Tuning Summary Table

| Rule | Version | False Positives | Tuning Applied | Planned Tuning |
|---|---|---|---|---|
| DET-01 PIM Outside Hours | v1.0 | 0 | None | Timezone adjustment if on-call noise |
| DET-02 GlobalAdmin Outside PIM | v1.0 | Break-glass renewal | Not suppressed — acknowledged | None |
| DET-03 MFA Disabled | v1.0 | 0 | InitiatedByUser != TargetUser filter | ModifiedProperty filter v1.1 |
| DET-04 CA Policy Modified | v1.0 | CA05 simulation | Not suppressed — expected | Change window suppression |
| DET-05 Impossible Travel | v1.0 | 0 | None | VPN watchlist exclusion v1.1 |
| DET-06 Azure Role Assignment | v1.0 | 0 | Owner/Contributor filter | System SP exclusion v1.1 |
| DET-07 Break-Glass Sign-In | v1.0 | Simulation test | Documented as test activity | Automation playbook |
| DET-08 Defender High Severity | v1.0 | 0 | None | Schema validation + SecurityAlert fallback |

---

## 4. Tuning Principles Applied

| Principle | How Applied |
|---|---|
| Start broad, tune narrow | Rules deployed with wider filters initially — narrow based on observed noise |
| Document every decision | Every false positive and tuning decision recorded in this log |
| Never suppress without documenting | All suppressions require rationale — no silent tuning |
| Test before deploying | All rules tested against historical data before deployment |
| Plan future tuning proactively | Known noise sources documented even before they fire |

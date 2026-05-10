# Attack Scenario

## Scenario Overview

This simulation models a realistic Microsoft 365 account compromise via phishing.
The attack is intentionally scoped to credential abuse and cloud-native exploitation,
reflecting one of the most common real-world identity attack patterns.

## Threat Actor Profile

| Attribute | Detail |
|---|---|
| **Type** | Opportunistic credential attacker |
| **Capability** | Valid credentials obtained via phishing |
| **Tools** | Browser-based OWA access only |
| **Goal** | Mailbox persistence and trust abuse for further phishing |

## Attacker Constraints (Simulation Scope)

- Valid username and password only - obtained via phishing
- No malware deployed
- No endpoint compromise
- No privilege escalation attempted
- No lateral movement to other accounts

## Why This Scenario

This scenario was chosen because it reflects the most common Microsoft 365 attack pattern
encountered in real SOC environments. Attackers increasingly rely on:

- Stolen credentials (no malware needed)
- Cloud-native features (inbox rules, OWA)
- Trusted internal identities (for further phishing)

This approach bypasses traditional endpoint security tools entirely, making identity
telemetry the only reliable detection source.

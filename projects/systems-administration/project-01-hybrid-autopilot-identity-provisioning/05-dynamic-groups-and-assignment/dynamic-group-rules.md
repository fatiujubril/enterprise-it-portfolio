# Dynamic Group Rules — Department-Driven Membership

## The concept

Phase 5 is where the `department` attribute — set once in on-premises Active Directory back in Phase 1, synced to Entra in Phase 2 — finally does work. Dynamic security groups evaluate that attribute and place users into department groups automatically, and those groups become the targeting mechanism for app assignment, configuration, and (later) Conditional Access. No one is added to these groups by hand; membership follows the data.

This is the **user track** of the provisioning model. The **device track** (the `AP-Devices-All` dynamic *device* group from Phase 4) is its counterpart — the two are built the same way but evaluate different object types and never intersect until a user signs in on a device at OOBE.

## The groups

Three dynamic **user** security groups were created, one per department:

| Group | Rule | Matches |
|---|---|---|
| `DYN-Users-Finance` | `(user.department -eq "Finance")` | Aisha Bello |
| `DYN-Users-Sales` | `(user.department -eq "Sales")` | Marcus Chen |
| `DYN-Users-IT` | `(user.department -eq "IT")` | Priya Sharma |

📸 [The three dynamic user groups created](./screenshots/01-dynamic-user-groups-created.png)

📸 [Finance dynamic membership rule](./screenshots/02-finance-dynamic-rule.png)

**Group type: Security** (not Microsoft 365) — Security groups are the assignment/targeting workhorse for apps, policy, and licensing; M365 groups are a collaboration construct and cannot even hold device members. **Membership type: Dynamic User** — distinct from the Dynamic Device flavor used for `AP-Devices-All`. The rule syntax differs by object type, and a `user.` rule in a device group (or vice versa) simply never matches.

The rules are deliberately simple single-attribute equalities. The interesting engineering is not the rule — it is the verification, and specifically the negative test.

## Positive verification

Each rule matched exactly its own department, with no cross-contamination:

📸 [Finance members — Aisha](./screenshots/03-finance-members-aisha.png)

📸 [Sales members — Marcus](./screenshots/04-sales-members-marcus.png)

📸 [IT members — Priya](./screenshots/05-it-members-priya.png)

Dynamic membership evaluates asynchronously — the groups are briefly empty after creation, so membership was confirmed before anything was assigned to them.

## The negative test — the point of John Doe

John Doe was created in Phase 1 deliberately without a department attribute, precisely for this moment. The test is not "does the rule match" (anyone can show that) but "does the rule correctly *not* match" — which is what proves the membership is genuinely attribute-driven rather than accidentally capturing everyone.

Cause and effect, as a pair:

📸 [John Doe — department attribute blank (the cause)](./screenshots/06-johndoe-department-blank.png)

📸 [John Doe — not a member of any DYN group (the effect)](./screenshots/07-johndoe-no-DYN-group-membership.png)

John's department is empty, so he matches none of the three rules and lands in none of the three groups — and therefore, once assignments exist, receives none of the department apps or policies. (He does appear in a directory-wide default group, which every user object joins simply by existing; that grants *presence*, not *access*, and does not dilute the test — the claim was never "John is in zero groups" but "John is in none of the attribute-driven assignment groups.")

### Why this matters as a principle

Membership is governed by an HR-sourced attribute, so **access follows the data**. If John's department is blank in AD, he gets nothing — which means data-quality problems surface as access problems, at the source. That is a feature, not a bug: it forces the fix where it belongs (the authoritative attribute in AD/HR) rather than papering over it with a manual group-add. This is the difference between attribute-driven access governance and manual group management, and it is exactly the kind of design an interviewer probes for.

## The pipeline behind the attribute

Worth stating explicitly, because it ties the whole build together: the `department` value these rules read did not originate in the cloud. It was set on the user object in on-premises AD (Phase 1), carried to Entra by Entra Connect whose sync scope deliberately includes the department attribute (Phase 2), and is now the root of trust for cloud access decisions (Phase 5). The on-premises directory remains the source of truth; the cloud acts on what it is told. Break the attribute at the source and every downstream decision changes — which is why sync hygiene and directory data quality are access-control concerns, not just administrative housekeeping.

---

[← Back to Phase 5 README](./README.md) | [Project overview](../README.md)

# Security Groups Design & Governance

## Purpose

This document defines the security group model used to support **Privileged Identity Management (PIM)**, **Conditional Access**, **Azure RBAC**, and **identity governance** within the tenant.

All privileged access is mediated through **security groups**, not direct user-to-role assignments.  
This design enables scalability, auditability, and alignment with Zero Trust and least-privilege principles.

---

## Design Principles

- No permanent privileged access for human administrator accounts
- Group-based access control over direct user assignment
- Clear separation between:
  - Identity governance
  - Role abstraction
  - Emergency access
- All privileged access is:
  - Just-in-time
  - Auditable
  - Reviewable

---

## Security Group Inventory

### 1. SEC-Admins
**Purpose:**  
Represents the security administration team responsible for day-to-day security operations.

- Members are **not** permanently privileged
- Privileged roles are accessed via PIM only

**Members:**
- `sec.admin`

**Evidence:**
- Group membership  
  `Evidence/Phase-01-Identity/Security-Groups/20260207_Group_SEC-Admins.png`

---

### 2. SEC-PIM-Eligible
**Purpose:**  
Defines which users are eligible to request privileged roles via Privileged Identity Management.

This group acts as the **identity governance gate** for privileged access.

**Members:**
- `sec.admin`

**Evidence:**
- Group membership  
  `Evidence/Phase-01-Identity/Security-Groups/20260207_Group_SEC-PIM-Eligible_Members.png`

---

### 3. SEC-CA-Admins
**Purpose:**  
Targets administrative accounts for enhanced Conditional Access enforcement.

Policies scoped to this group enforce:
- Mandatory MFA
- Strong authentication requirements
- Protection against risky sign-ins

**Members:**
- `sec.admin`

**Explicit Exclusions:**
- Break-glass accounts are intentionally excluded

**Evidence:**
- Group membership  
  `Evidence/Phase-01-Identity/Security-Groups/20260207_Group_SEC-CA-Admins_Members.png`

---

### 4. SEC-BreakGlass
**Purpose:**  
Contains emergency access accounts used only for tenant recovery scenarios.

Break-glass accounts:
- Retain permanent Global Administrator privileges
- Are excluded from Conditional Access and PIM
- Are protected with long, complex passwords stored offline

**Members:**
- `bg.admin1`
- `bg.admin2`

**Evidence:**
- Group membership  
  `Evidence/Phase-01-Identity/Security-Groups/20260207_Group_SEC-BreakGlass_Members.png`

---

### 5. SEC-Global-Admins
**Purpose:**  
Acts as a **role abstraction layer** for the Global Administrator role.

This group is intentionally kept **empty** and exists to support:
- Group-based role assignment
- Scalable privilege management
- Clear separation between identity eligibility and role assignment

No users are directly assigned to Global Administrator.

**Members:**
- None (by design)

**Evidence:**
- Group membership  
  `Evidence/Phase-01-Identity/Security-Groups/20260207_Group_SEC-Global-Admins_Empty.png`

---

### 6. SEC-Azure-Contributors
**Purpose:**  
Used for Azure Role-Based Access Control (RBAC) assignments following least-privilege principles.

This group will be assigned Azure roles instead of individual users.

**Members:**
- `sec.admin`

**Evidence:**
- Group membership  
  `Evidence/Phase-01-Identity/Security-Groups/20260207_Group_SEC-Azure-Contributors.png`

---

## Governance & Audit Considerations

- All group memberships are explicitly assigned and documented
- Privileged role access is:
  - Time-bound
  - Justified
  - Logged
- Group-based access supports:
  - Periodic access reviews
  - Incident investigations
  - Compliance audits (ISO 27001 / NIST)

---

## Summary

This security group model establishes a strong foundation for:
- Privileged Identity Management
- Conditional Access enforcement
- Azure RBAC
- Zero Trust identity architecture

It ensures that **no standing human privilege exists** while preserving secure emergency access and full auditability.

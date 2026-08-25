# Security Control Mapping

## 1. Purpose

This document maps the technical capabilities demonstrated in the project to common security-control objectives.

The mapping is intended for portfolio, learning, and technical demonstration purposes.

It is **not a formal compliance assessment**.

---

# 2. Control Mapping

| Security Objective        | Project Implementation                            | Evidence                     |
| ------------------------- | ------------------------------------------------- | ---------------------------- |
| Account Management        | Retrieve and analyze Entra ID users               | User discovery output        |
| Identity Lifecycle        | Detect inactive identities                        | Stale account report         |
| Access Review             | Identify potentially unnecessary active accounts  | Stale account detection      |
| Privileged Access Review  | Cross-reference stale users with directory roles  | Privileged membership output |
| Least Privilege           | Identify unnecessary privileged exposure          | Privileged account analysis  |
| Change Control            | Perform dry run before remediation                | Dry-run output               |
| Account Remediation       | Disable qualifying internal accounts              | Remediation output           |
| Audit Logging             | Record remediation locally                        | `remediation-log.csv`        |
| Independent Logging       | Validate actions against Entra audit logs         | Entra audit evidence         |
| Verification              | Query account state after modification            | `AccountEnabled = False`     |
| Administrative Protection | Exclude authenticated administrator               | Safety-filter logic          |
| Guest Protection          | Exclude external guest from automated remediation | Guest exclusion              |
| Evidence Generation       | Export findings to CSV                            | Stale-account report         |

---

# 3. ISO/IEC 27001 Control Alignment

The project supports access-control and logging objectives associated with ISO/IEC 27001.

| Area                | Demonstrated Capability                         |
| ------------------- | ----------------------------------------------- |
| Identity management | Automated user discovery                        |
| Access rights       | Stale-account analysis                          |
| Privileged access   | Directory role membership check                 |
| Access review       | Automated identification of inactive identities |
| Logging             | Local and Entra audit records                   |
| Change management   | Dry-run before remediation                      |

The project does not claim ISO/IEC 27001 certification.

---

# 4. NIST CSF Mapping

| NIST CSF Function | Project Activity                                      |
| ----------------- | ----------------------------------------------------- |
| Identify          | Discover users and access relationships               |
| Protect           | Disable unnecessary active accounts                   |
| Detect            | Detect stale identities                               |
| Respond           | Automatically remediate qualifying accounts           |
| Recover           | Verify resulting identity state and preserve evidence |

---

# 5. CIS Control Alignment

| CIS Objective             | Project Capability                         |
| ------------------------- | ------------------------------------------ |
| Account Management        | User inventory and stale-account detection |
| Access Control Management | Privileged membership analysis             |
| Audit Log Management      | Entra audit-log validation                 |
| Least Privilege           | Privileged-account risk analysis           |
| Secure Configuration      | Controlled identity remediation            |

---

# 6. IAM Control Lifecycle

The project implements a simplified identity-control lifecycle:

```text
DISCOVER
   ↓
CLASSIFY
   ↓
REVIEW
   ↓
APPROVE / DRY RUN
   ↓
REMEDIATE
   ↓
VERIFY
   ↓
AUDIT
```

This model can be expanded into a production access-certification process involving managers, application owners, security teams, and compliance teams.

---

# 7. Evidence Mapping

## Control: Identity Discovery

Evidence:

```text
Get-MgUser
```

Purpose:

Establish the population of identities being evaluated.

---

## Control: Stale Account Detection

Evidence:

```text
stale-accounts-report-YYYY-MM-DD.csv
```

Purpose:

Document identities classified as stale according to the laboratory policy.

---

## Control: Privileged Access Review

Evidence:

```text
$stalePrivileged.Count
```

Result:

```text
0
```

Purpose:

Demonstrate that no stale Global Administrator accounts were identified.

---

## Control: Change Review

Evidence:

```text
[DRY RUN] Would disable: ...
```

Purpose:

Demonstrate that remediation targets were reviewed before modification.

---

## Control: Account Remediation

Evidence:

```text
Disabled: lindiwe@sage484.onmicrosoft.com
Disabled: sarah@sage484.onmicrosoft.com
Disabled: sipho@sage484.onmicrosoft.com
```

Purpose:

Demonstrate automated account lifecycle enforcement.

---

## Control: Verification

Evidence:

```text
lindiwe@sage484.onmicrosoft.com: AccountEnabled = False
sarah@sage484.onmicrosoft.com: AccountEnabled = False
sipho@sage484.onmicrosoft.com: AccountEnabled = False
```

Purpose:

Confirm that the intended directory state was achieved.

---

## Control: Independent Audit Verification

Evidence:

```text
Disable account
```

recorded in Microsoft Entra audit logs.

Purpose:

Provide an independent system-of-record confirmation of the remediation.

---

# 8. Control Maturity

The project demonstrates a progression beyond manual identity administration.

### Level 1 — Manual

Administrator manually reviews users and disables accounts.

### Level 2 — Detection

Script identifies potentially stale accounts.

### Level 3 — Reporting

Script generates an audit-ready report.

### Level 4 — Controlled Remediation

Script performs dry-run validation and automated remediation.

### Level 5 — Verified Automation

Remediation is independently validated through Microsoft Entra audit logs.

This laboratory implementation demonstrates capabilities across **Levels 4–5**.

---

# 9. Production Control Recommendations

Before using a similar workflow in production, organizations should add:

* Formal access-review approval
* Service-account exclusions
* Break-glass account protection
* Guest lifecycle policies
* Ticket/change-management integration
* Centralized logging
* Alerting
* Automated rollback
* Managed identity authentication
* Privileged execution controls
* Scheduled execution
* Exception management
* Evidence retention requirements

---

# 10. Final Assessment

The project demonstrates that Microsoft Graph and PowerShell can be used to implement a controlled identity-governance workflow that:

* Discovers identities
* Detects stale access
* Evaluates privilege exposure
* Produces evidence
* Applies safety controls
* Performs remediation
* Verifies the resulting state
* Maintains an independent audit trail

The implementation should be considered a **technical control demonstration**, not a certification or formal compliance assessment.

# Automated Access Certification & Stale Account Remediation

## Overview

This project demonstrates an automated identity governance and account remediation workflow using Microsoft Graph PowerShell and Microsoft Entra ID.

The objective was to replace a manual stale-account review process with a repeatable workflow that can:

* Retrieve users and sign-in activity from Microsoft Entra ID
* Identify enabled accounts with no recorded interactive sign-in activity
* Cross-reference stale accounts against privileged role membership
* Generate an audit-ready CSV report
* Perform a dry run before making changes
* Exclude protected administrative identities from automated remediation
* Disable qualifying internal accounts through Microsoft Graph
* Generate a local remediation audit trail
* Verify the resulting account state
* Independently validate remediation through Microsoft Entra audit logs

This project focuses on the operational side of IAM rather than portal-only configuration.

---

## Business Problem

Identity teams regularly need to answer questions such as:

* Which enabled accounts are no longer being used?
* Are inactive accounts still holding privileged access?
* Which identities should be reviewed or remediated?
* Can stale accounts be identified consistently?
* Can remediation actions be performed safely and recorded for audit purposes?

Manually answering these questions becomes increasingly difficult as an organization's identity population grows.

This project demonstrates how Microsoft Graph can be used to automate the detection, analysis, reporting, and remediation stages of that process.

---

## Architecture

```text
                    Microsoft Entra ID
                           |
                           |
                    Microsoft Graph
                           |
                           v
                +----------------------+
                | PowerShell Automation|
                +----------------------+
                           |
             +-------------+-------------+
             |                           |
             v                           v
      User + Sign-In Data        Directory Role Data
             |                           |
             +-------------+-------------+
                           |
                           v
                  Stale Account Detection
                           |
                           v
                  Privileged Account Check
                           |
                           v
                     Safety Filtering
                           |
                           v
                       Dry Run
                           |
                           v
                    Remediation
                           |
              +------------+------------+
              |                         |
              v                         v
       Local Audit Log          Entra Audit Logs
              |                         |
              +------------+------------+
                           |
                           v
                     Verification
```

See the detailed architecture documentation in [`architecture/architecture.md`](architecture/architecture.md).

---

## Technology Stack

| Technology                     | Purpose                               |
| ------------------------------ | ------------------------------------- |
| Microsoft Entra ID             | Identity directory and access control |
| Microsoft Graph                | Identity data and management API      |
| Microsoft Graph PowerShell SDK | Automation interface                  |
| PowerShell 7                   | Automation and remediation            |
| CSV                            | Audit/reporting output                |
| Entra Audit Logs               | Independent verification              |

---

## Required Microsoft Graph Permissions

The lab used delegated Microsoft Graph permissions for:

```text
User.Read.All
AuditLog.Read.All
Directory.Read.All
RoleManagement.Read.Directory
User.ReadWrite.All
```

These permissions were used for identity retrieval, audit investigation, role membership analysis, and account remediation.

In a production implementation, permissions should be reduced to the minimum required by the automation architecture.

---

## Workflow

### 1. Authentication

The automation authenticates to Microsoft Graph using delegated access.

The Graph context was verified before performing identity operations.

### 2. Identity Discovery

The automation retrieves users with:

* User ID
* Display name
* User Principal Name
* Account status
* Sign-in activity

### 3. Stale Account Detection

The lab policy identifies enabled accounts that have no recorded interactive sign-in activity.

The detection logic also supports a 90-day inactivity threshold when a sign-in timestamp is available.

### 4. Privileged Access Analysis

Detected stale accounts are cross-referenced against Global Administrator membership.

The lab produced:

```text
Stale accounts detected: 4
Stale Global Administrators: 0
```

No stale Global Administrator account was identified.

### 5. Reporting

The automation exports the findings to:

```text
stale-accounts-report-YYYY-MM-DD.csv
```

The report includes:

* Display name
* User Principal Name
* Account status
* Last recorded sign-in
* Privileged status

### 6. Dry Run

Before remediation, the automation performs a dry run showing which accounts would be disabled.

This provides an opportunity to review the proposed changes before modifying identities.

### 7. Safety Filtering

The authenticated administrative account was explicitly excluded from remediation.

External guest accounts were also excluded from the automatic remediation workflow.

### 8. Remediation

Three internal stale accounts were disabled through Microsoft Graph:

```text
lindiwe@sage484.onmicrosoft.com
sarah@sage484.onmicrosoft.com
sipho@sage484.onmicrosoft.com
```

### 9. Verification

Each remediated account was queried after the operation.

The resulting state was:

```text
AccountEnabled = False
```

for all three remediated accounts.

### 10. Independent Audit Verification

Microsoft Entra audit logs independently recorded the remediation operations.

The audit events showed:

```text
Disable account
```

for all three remediated accounts.

The audit records also identified the initiating administrator.

---

## Evidence

The project produced several forms of evidence:

### Detection Evidence

Evidence showing the accounts identified by the stale-account logic.

### Privileged Access Evidence

Evidence showing that no stale Global Administrator accounts were identified.

### Report Evidence

The generated CSV report containing the stale-account findings.

### Dry-Run Evidence

PowerShell output showing which accounts would be remediated before execution.

### Remediation Evidence

PowerShell output showing the accounts that were disabled.

### Verification Evidence

Post-remediation queries showing:

```text
AccountEnabled = False
```

### Audit Evidence

Microsoft Entra audit log entries independently confirming the account-disable operations.

See [`screenshots/README.md`](screenshots/README.md) for the evidence index.

---

## Compliance Alignment

The project demonstrates controls relevant to common identity governance and security frameworks, including:

* ISO/IEC 27001
* NIST Cybersecurity Framework
* CIS Controls
* Least Privilege
* Identity Lifecycle Management
* Access Review
* Privileged Access Management
* Audit Logging

The project does **not** claim formal compliance or certification.

The compliance documentation describes how the technical controls demonstrated in this lab can support broader organizational control objectives.

See [`compliance/compliance.md`](compliance/compliance.md).

---

## Security Policies

The project includes documented policies covering:

* Stale account identification
* Privileged access
* Automated remediation
* Safety exclusions
* Audit logging

See the [`policies/`](policies/) directory.

---

## Limitations

This is a controlled laboratory implementation and should not be deployed directly into production without additional safeguards.

Important limitations include:

1. Blank sign-in activity was treated as stale according to the lab policy.
2. Guest accounts were excluded from automatic remediation.
3. The remediation workflow was manually executed rather than scheduled.
4. Global Administrator membership was used as the privileged-access test case.
5. The automation uses delegated permissions.
6. Break-glass accounts require explicit exclusion in a production implementation.
7. Service accounts require separate lifecycle policies.
8. Production environments should include approval workflows and change-management controls.
9. Additional logging and centralized monitoring should be implemented for production use.

---

## Production Improvements

A production implementation could introduce:

* Microsoft Entra managed identities
* Azure Automation or Azure Functions
* Application permissions with tightly scoped access
* Microsoft Entra PIM
* Access Reviews
* Approval workflows
* Microsoft Sentinel integration
* Centralized logging
* Service-account exclusions
* Break-glass account protection
* Guest lifecycle governance
* Automated ticket creation
* Rollback procedures
* Alerting and monitoring
* Scheduled execution

---

## Project Outcome

The final workflow demonstrates that identity governance can move beyond manual portal operations into repeatable automation.

The project successfully demonstrated:

```text
Identity Discovery
        ↓
Stale Account Detection
        ↓
Privilege Analysis
        ↓
Safety Filtering
        ↓
Dry Run
        ↓
Automated Remediation
        ↓
Local Audit Logging
        ↓
State Verification
        ↓
Entra Audit Verification
```

The key outcome is not simply that accounts were disabled.

The project demonstrates a controlled identity remediation lifecycle where every major action is:

* Detectable
* Reviewable
* Automated
* Logged
* Verifiable
* Auditable

---

## Repository Structure

```text
.
├── README.md
│
├── architecture/
│   ├── architecture.md
│   └── identity-remediation-flow.md
│
├── compliance/
│   ├── compliance.md
│   └── control-mapping.md
│
├── policies/
│   ├── stale-account-policy.md
│   ├── privileged-access-policy.md
│   └── remediation-policy.md
│
└── screenshots/
    ├── README.md
    ├── 01-graph-authentication.png
    ├── 02-stale-account-detection.png
    ├── 03-privileged-account-check.png
    ├── 04-audit-report.png
    ├── 05-dry-run.png
    ├── 06-remediation.png
    ├── 07-verification.png
    └── 08-entra-audit-log.png
```

---

## Author

**Kananelo Mohale**

Focus areas:

* Identity & Access Management
* Microsoft Entra ID
* Cloud Security
* Privileged Access Management
* Identity Governance
* Security Automation

---

## Disclaimer

This repository represents a controlled laboratory implementation designed for educational and portfolio purposes.

Account remediation actions were performed against a test tenant. Production identity environments require additional governance, approval, testing, monitoring, rollback procedures, and change-management controls.

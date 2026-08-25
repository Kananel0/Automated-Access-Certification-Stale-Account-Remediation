# Architecture — Automated Access Certification & Stale Account Remediation

## 1. Architecture Overview

This project implements an identity governance and automated remediation workflow using Microsoft Entra ID, Microsoft Graph, and PowerShell.

The architecture is designed around a simple principle:

> Detect identity risk → evaluate access → apply safety controls → remediate → verify → audit.

Instead of manually reviewing individual accounts through the Microsoft Entra portal, the automation retrieves identity data programmatically and applies a defined remediation policy.

---

## 2. High-Level Architecture

```text
                           +----------------------+
                           |   Microsoft Entra ID |
                           |                      |
                           | Users                |
                           | Sign-In Activity     |
                           | Directory Roles      |
                           | Audit Logs            |
                           +----------+-----------+
                                      |
                                      |
                              Microsoft Graph
                                      |
                                      v
                    +-------------------------------+
                    | Microsoft Graph PowerShell SDK |
                    +---------------+---------------+
                                    |
                                    v
                         +----------------------+
                         | PowerShell Automation|
                         +----------+-----------+
                                    |
                +-------------------+-------------------+
                |                   |                   |
                v                   v                   v
        User Discovery      Role Membership       Audit Data
                |                   |                   |
                +-------------------+-------------------+
                                    |
                                    v
                         +----------------------+
                         | Stale Account Engine |
                         +----------+-----------+
                                    |
                                    v
                         +----------------------+
                         | Privilege Evaluation |
                         +----------+-----------+
                                    |
                                    v
                         +----------------------+
                         | Safety Controls      |
                         |                      |
                         | Admin Exclusion      |
                         | Guest Exclusion      |
                         | Dry Run              |
                         +----------+-----------+
                                    |
                                    v
                         +----------------------+
                         | Remediation Engine   |
                         |                      |
                         | Disable Account      |
                         +----------+-----------+
                                    |
                     +--------------+--------------+
                     |                             |
                     v                             v
              Local CSV Log                Entra Audit Log
                     |                             |
                     +--------------+--------------+
                                    |
                                    v
                         +----------------------+
                         | Verification         |
                         | AccountEnabled=False |
                         +----------------------+
```

---

## 3. Core Components

### 3.1 Microsoft Entra ID

Microsoft Entra ID acts as the identity system of record.

The automation interacts with Entra ID to retrieve:

* User identities
* Account status
* Sign-in activity
* Directory role membership
* Audit events

Entra ID also records the resulting remediation operations in its audit logs.

---

### 3.2 Microsoft Graph

Microsoft Graph provides the API layer between the PowerShell automation and Microsoft Entra ID.

The project uses Graph to perform identity lifecycle operations rather than relying exclusively on portal-based administration.

The automation uses Graph operations for:

* User retrieval
* Sign-in activity retrieval
* Directory role membership retrieval
* Account updates
* Audit log retrieval

---

### 3.3 Microsoft Graph PowerShell SDK

The Microsoft Graph PowerShell SDK provides PowerShell cmdlets for interacting with Microsoft Graph.

The lab used modules including:

```text
Microsoft.Graph
Microsoft.Graph.Users
Microsoft.Graph.Reports
```

This allowed the project to perform both identity management and audit investigation from a PowerShell environment.

---

### 3.4 PowerShell Automation

PowerShell acts as the automation layer.

The workflow performs:

1. Authentication
2. User discovery
3. Sign-in analysis
4. Stale-account classification
5. Privileged-access analysis
6. Report generation
7. Dry-run simulation
8. Safety filtering
9. Account remediation
10. Verification
11. Audit validation

---

## 4. Identity Data Flow

The identity data flows through several processing stages.

```text
Microsoft Entra Users
        |
        v
User + Sign-In Activity
        |
        v
Stale Account Detection
        |
        v
Privileged Role Cross-Reference
        |
        v
Safety Filtering
        |
        v
Remediation Decision
        |
        v
Account Update
```

The automation does not immediately modify accounts after discovering them.

A staged process is used to reduce the risk of unintended changes.

---

## 5. Stale Account Detection

The detection engine evaluates enabled users against the defined inactivity policy.

The primary conditions are:

```text
AccountEnabled = True
AND
LastSignIn < 90 days
OR
No recorded interactive sign-in activity
```

In this laboratory implementation, accounts with no recorded interactive sign-in activity were classified as stale.

This behavior is intentionally documented because a blank sign-in timestamp does not necessarily prove that an identity has never authenticated.

---

## 6. Privileged Access Evaluation

Stale accounts are cross-referenced against directory role membership.

The lab specifically evaluated Global Administrator membership.

```text
Stale User
    |
    v
Global Administrator?
    |
   / \
 Yes  No
 |     |
 v     v
High   Standard
Risk   Stale Account
```

The lab result was:

```text
Stale Accounts: 4
Stale Global Administrators: 0
```

No stale Global Administrator account was found.

---

## 7. Safety Controls

The remediation architecture includes multiple safety controls.

### Administrative Account Exclusion

The currently authenticated administrator was explicitly excluded from remediation.

```text
mananasefadi@sage484.onmicrosoft.com
```

### Guest Account Exclusion

External guest identities were excluded from the automatic remediation workflow.

The detected guest account was therefore not automatically disabled.

### Dry Run

A dry-run stage displays the proposed remediation targets before changes are executed.

### Verification

Every remediated account is queried after the update to confirm the resulting account state.

---

## 8. Remediation Architecture

Only accounts that pass the detection and safety checks are passed to the remediation engine.

```text
Stale Account
      |
      v
Safety Checks
      |
      +---- Administrator? ----> EXCLUDE
      |
      +---- Guest? ------------> EXCLUDE
      |
      v
Internal Account
      |
      v
Remediation
      |
      v
AccountEnabled = False
```

Three internal test accounts were successfully remediated.

---

## 9. Logging Architecture

The project uses two separate evidence sources.

### Local Remediation Log

The PowerShell script records:

* Timestamp
* User
* Action
* Reason

Example:

```text
Timestamp,User,Action,Reason
2026-08-25T22:53:51,lindiwe@sage484.onmicrosoft.com,AccountDisabled,NoInteractiveSignInActivity
```

### Microsoft Entra Audit Log

Entra ID independently records the identity modification.

The audit records included:

```text
Activity: Disable account
Target: lindiwe@sage484.onmicrosoft.com
InitiatedBy: mananasefadi@sage484.onmicrosoft.com
```

This provides independent verification of the remediation.

---

## 10. Verification Architecture

After remediation, the automation queries each account again.

Expected result:

```text
AccountEnabled = False
```

The three remediated accounts returned:

```text
lindiwe@sage484.onmicrosoft.com → False
sarah@sage484.onmicrosoft.com   → False
sipho@sage484.onmicrosoft.com   → False
```

The workflow therefore verifies the actual directory state rather than relying solely on the success of the update command.

---

## 11. Security Design Principles

The architecture demonstrates several IAM security principles:

### Least Privilege

Only required Microsoft Graph permissions should be granted to an automation identity.

### Separation of Detection and Remediation

Finding an account does not immediately result in modification.

### Explicit Safety Controls

Administrative and guest identities are protected from automated changes.

### Auditability

Both local and Microsoft Entra audit records are maintained.

### Verification

Changes are independently checked after execution.

### Repeatability

The workflow can be executed repeatedly against the identity population.

---

## 12. Production Architecture Considerations

A production implementation should replace the manually executed PowerShell workflow with a controlled automation platform.

A possible production architecture would be:

```text
                 Microsoft Entra ID
                         |
                         v
                Microsoft Graph
                         |
                         v
              Azure Automation /
              Azure Function
                         |
             +-----------+-----------+
             |                       |
             v                       v
      Detection Engine        Policy Engine
             |                       |
             +-----------+-----------+
                         |
                         v
                  Approval Workflow
                         |
                         v
                    Remediation
                         |
             +-----------+-----------+
             |                       |
             v                       v
       Microsoft Sentinel      Entra Audit Logs
```

Potential production improvements include:

* Managed identities
* Centralized logging
* Microsoft Sentinel integration
* Approval workflows
* Service-account exclusions
* Break-glass account protection
* Guest lifecycle policies
* Ticket integration
* Automated rollback
* Scheduled execution
* Alerting

---

## 13. Architecture Outcome

The architecture demonstrates a complete identity remediation lifecycle:

```text
DISCOVER
   ↓
ANALYZE
   ↓
CLASSIFY
   ↓
PROTECT
   ↓
DRY RUN
   ↓
REMEDIATE
   ↓
VERIFY
   ↓
AUDIT
```

The key architectural objective is to transform identity governance from a manual administrative activity into a controlled, repeatable, auditable process.

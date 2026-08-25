# Identity Remediation Flow

## Purpose

This document describes the end-to-end decision flow used by the automated stale-account remediation process.

The workflow is designed to prevent immediate destructive action after an account is identified.

---

## 1. End-to-End Flow

```text
START
  |
  v
Authenticate to Microsoft Graph
  |
  v
Retrieve Enabled Users
  |
  v
Retrieve Sign-In Activity
  |
  v
Identify Stale Accounts
  |
  v
Check Privileged Role Membership
  |
  v
Generate Report
  |
  v
Dry Run
  |
  v
Apply Safety Exclusions
  |
  +---- Administrator? ---- YES ---> EXCLUDE
  |
  +---- Guest? ------------ YES ---> EXCLUDE
  |
  v
Internal Stale Account
  |
  v
Disable Account
  |
  v
Write Remediation Log
  |
  v
Verify Account State
  |
  v
Query Entra Audit Logs
  |
  v
END
```

---

## 2. Detection Decision

The automation evaluates every enabled account.

```text
Is account enabled?
       |
      YES
       |
       v
Is sign-in activity older than policy threshold
OR is there no recorded interactive sign-in activity?
       |
      YES
       |
       v
Classify as stale
```

---

## 3. Privileged Access Decision

Every stale account is checked against privileged role membership.

```text
Stale Account
     |
     v
Privileged Role?
   /       \
 YES       NO
 |          |
 v          v
High-risk  Standard
finding    stale account
```

In this lab:

```text
Stale accounts = 4
Stale Global Administrators = 0
```

---

## 4. Safety Decision

Before remediation:

```text
Is identity the authenticated administrator?
          |
         YES
          |
          v
       EXCLUDE
```

Then:

```text
Is identity an external guest?
          |
         YES
          |
          v
       EXCLUDE
```

Only identities that pass both checks proceed to remediation.

---

## 5. Dry Run

The automation first displays the proposed actions:

```text
[DRY RUN] Would disable: user@example.com
```

No account changes occur during this stage.

The operator can therefore review the proposed remediation set before execution.

---

## 6. Remediation

After safety filtering, eligible internal stale accounts are disabled through Microsoft Graph.

The lab remediated:

```text
lindiwe@sage484.onmicrosoft.com
sarah@sage484.onmicrosoft.com
sipho@sage484.onmicrosoft.com
```

---

## 7. Verification

After remediation, each account is queried again.

```text
AccountEnabled = False
```

This confirms the desired directory state.

---

## 8. Independent Audit Validation

The workflow then queries Microsoft Entra audit logs.

Expected event:

```text
ActivityDisplayName = Disable account
```

The audit event provides an independent record of the modification.

---

## 9. Final Control Flow

```text
             DETECT
                |
                v
             ANALYZE
                |
                v
          CHECK PRIVILEGE
                |
                v
           GENERATE REPORT
                |
                v
             DRY RUN
                |
                v
          SAFETY FILTER
                |
                v
           REMEDIATE
                |
                v
             VERIFY
                |
                v
          AUDIT VALIDATE
                |
                v
             COMPLETE
```

This flow provides a controlled identity lifecycle where remediation is based on defined conditions and every executed change can be traced and independently verified.

# Compliance Alignment

## 1. Purpose

This document explains how the Automated Access Certification & Stale Account Remediation project aligns with common identity, access-control, and security governance objectives.

The project demonstrates technical controls for:

* Identity lifecycle management
* Access governance
* Privileged access monitoring
* Stale account detection
* Automated remediation
* Least privilege
* Audit logging
* Change verification
* Security monitoring

This repository **does not claim formal compliance, certification, or audit attestation** against any framework.

The mappings below demonstrate how the technical controls implemented in this laboratory can support broader organizational compliance objectives.

---

# 2. Control Objectives

The project is designed around the following security objectives:

| Objective                     | Implementation                             |
| ----------------------------- | ------------------------------------------ |
| Identify inactive identities  | Microsoft Graph sign-in activity           |
| Review privileged access      | Directory role membership analysis         |
| Reduce unnecessary access     | Automated account disabling                |
| Protect administrators        | Explicit administrator exclusion           |
| Protect guest identities      | Guest exclusion from automatic remediation |
| Prevent unsafe changes        | Dry-run process                            |
| Maintain evidence             | CSV report and remediation log             |
| Verify changes                | Post-remediation account-state check       |
| Maintain independent evidence | Microsoft Entra audit logs                 |

---

# 3. ISO/IEC 27001 Alignment

The project can support identity and access-control objectives associated with an ISO/IEC 27001 Information Security Management System.

Relevant areas include:

### Identity Management

The automation identifies user accounts that may no longer require active access.

This supports the broader objective of maintaining controlled identity lifecycles.

### Access Rights

The project evaluates whether stale identities remain associated with privileged directory roles.

This supports periodic review of access rights.

### Privileged Access

The automation specifically cross-references stale accounts against Global Administrator membership.

The laboratory result was:

```text
Stale Accounts: 4
Stale Global Administrators: 0
```

### Logging

The remediation process creates a local audit trail and Microsoft Entra independently records the account modifications.

### Change Control

The dry-run stage provides a review point before account modifications are performed.

---

# 4. NIST Cybersecurity Framework Alignment

The project aligns conceptually with several NIST Cybersecurity Framework functions.

## Identify

The automation identifies:

* Active user accounts
* Sign-in activity
* Potentially stale identities
* Privileged role membership

## Protect

The remediation process reduces unnecessary active identities by disabling accounts classified as stale.

Administrative identities are explicitly excluded from automated remediation.

## Detect

The stale-account detection logic identifies identities that meet the defined inactivity criteria.

## Respond

The remediation engine automatically disables qualifying internal accounts.

The action is recorded in a local remediation log.

## Recover

The verification and audit process provides evidence of what changed and when.

A production implementation could extend this with automated rollback procedures.

---

# 5. CIS Controls Alignment

The project also supports identity-related objectives commonly addressed by the CIS Controls.

### Account Management

The project identifies enabled accounts that may no longer be required.

### Access Control Management

The workflow evaluates privileged role membership for stale accounts.

### Audit Log Management

The project uses Microsoft Entra audit logs to independently validate account modifications.

### Least Privilege

The workflow treats stale privileged accounts as a higher-risk condition and uses explicit safety controls around administrative identities.

---

# 6. Identity Governance Alignment

The project demonstrates several core identity governance principles.

### Joiner

New identities should receive only the access required for their role.

### Mover

Changes in role or responsibility should trigger access review.

### Leaver

Inactive or departed identities should eventually lose access.

This project focuses primarily on the **leaver and dormant-account governance problem**.

---

# 7. Privileged Access Management

Privileged access represents a higher-risk category of identity exposure.

The project therefore performs an additional privilege check after identifying stale identities.

The workflow is:

```text
Stale Account
      |
      v
Privileged Role Check
      |
      v
Risk Classification
```

Although the lab specifically evaluated Global Administrator membership, the same architecture could be extended to:

* User Administrator
* Privileged Role Administrator
* Security Administrator
* Exchange Administrator
* SharePoint Administrator
* Other sensitive directory roles

---

# 8. Auditability

The project produces two independent forms of evidence.

## Local Evidence

The PowerShell remediation log records:

```text
Timestamp
User
Action
Reason
```

Example:

```text
2026-08-25T22:53:51
lindiwe@sage484.onmicrosoft.com
AccountDisabled
NoInteractiveSignInActivity
```

## Microsoft Entra Evidence

Microsoft Entra audit logs independently recorded:

```text
Activity: Disable account
Target: lindiwe@sage484.onmicrosoft.com
InitiatedBy: mananasefadi@sage484.onmicrosoft.com
```

This separation provides stronger evidence than relying exclusively on the automation's own log.

---

# 9. Separation of Detection and Remediation

The workflow deliberately separates identification from modification.

```text
Detection
   |
   v
Analysis
   |
   v
Report
   |
   v
Dry Run
   |
   v
Safety Review
   |
   v
Remediation
```

This reduces the risk of immediately modifying an incorrectly identified identity.

---

# 10. Least Privilege Considerations

The laboratory used delegated Microsoft Graph permissions required to perform the workflow.

Permissions included:

```text
User.Read.All
AuditLog.Read.All
Directory.Read.All
RoleManagement.Read.Directory
User.ReadWrite.All
```

In production, these permissions should be reviewed carefully and reduced wherever possible.

A production automation identity should use:

* Minimum required permissions
* Controlled credentials
* Managed identities where appropriate
* Restricted execution environments
* Monitoring
* Administrative approval

---

# 11. Compliance Evidence Produced

The project generated the following evidence:

| Evidence                     | Purpose                               |
| ---------------------------- | ------------------------------------- |
| Graph connection output      | Authentication evidence               |
| User/sign-in activity output | Identity discovery                    |
| Stale account report         | Detection evidence                    |
| Privileged role check        | Access-risk evidence                  |
| Dry-run output               | Change-control evidence               |
| Remediation log              | Local audit evidence                  |
| Account verification         | Technical validation                  |
| Entra audit log              | Independent system-of-record evidence |

---

# 12. Compliance Limitations

This project should not be interpreted as proof of organizational compliance.

A real compliance assessment would also require evaluation of:

* Governance
* Policies
* Procedures
* Risk management
* Human processes
* Change management
* Access approval
* Incident response
* Monitoring
* Evidence retention
* Organizational responsibilities
* Internal and external audit requirements

The laboratory demonstrates technical capabilities that can contribute to those broader control objectives.

---

# 13. Conclusion

The project demonstrates how identity governance controls can be implemented through automation rather than relying exclusively on manual administration.

The resulting workflow provides:

```text
Identity Discovery
       ↓
Risk Detection
       ↓
Privilege Analysis
       ↓
Controlled Remediation
       ↓
Audit Evidence
       ↓
Independent Verification
```

This provides a practical foundation for identity governance, access certification, privileged access monitoring, and automated lifecycle management.

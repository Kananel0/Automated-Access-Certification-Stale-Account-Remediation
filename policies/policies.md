# Identity Governance Policies

## 1. Purpose

This directory contains the identity governance policies used by the Automated Access Certification & Stale Account Remediation project.

The policies define the conditions under which an identity is considered stale, how privileged identities are evaluated, and what safeguards must be applied before automated remediation.

These policies represent the rules implemented in the laboratory environment.

---

## 2. Policy Objectives

The policy framework is designed to:

- Reduce unnecessary active accounts
- Detect dormant identities
- Identify stale privileged accounts
- Protect administrative identities
- Prevent unsafe automated changes
- Maintain an audit trail
- Verify remediation actions
- Support repeatable identity governance

---

## 3. Policy Hierarchy

```text
Identity Governance Policy
          |
          +-------------------------+
          |                         |
          v                         v
 Stale Account Policy       Privileged Access Policy
          |                         |
          +------------+------------+
                       |
                       v
              Remediation Policy
                       |
                       v
                PowerShell Automation
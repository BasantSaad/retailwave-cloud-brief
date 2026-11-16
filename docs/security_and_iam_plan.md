# Cloud Fundamentals & Security Plan

**Client:** RetailWave Analytics
**Prepared by:** ApexCloud Solutions
**Target Audience:** Elena Rostova (VP of Engineering) & Executive Leadership

*Context: Following the recent 48-hour outage caused by a localized power failure at the Chicago data center, this document outlines the foundational security, identity, and governance strategies required for a secure transition to AWS.*

---

## 1. AWS Shared Responsibility Model: Concept Summary

The AWS Shared Responsibility Model divides security duties between AWS and the customer. In the move from **On-Premises** infrastructure to AWS, RetailWave stops owning the physical layer but keeps ownership of who can access its systems and what happens to its data.

### Security OF the Cloud (AWS responsibility)

AWS protects the infrastructure that runs all AWS services:

- Physical data centers: buildings, power, cooling, and physical access control, including each **Availability Zone (AZ)** in an **AWS Region**
- Hardware: servers, storage devices, and networking equipment, including hardware replacement and disposal
- The global network, and the virtualization layer that isolates one customer from another in a multi-tenant environment
- The underlying software of managed services (for **PaaS** and **SaaS** offerings, this includes the operating system and platform patching)

### Security IN the Cloud (RetailWave's responsibility)

RetailWave is responsible for everything it puts into and configures within AWS:

- **IAM:** creating users, groups, roles, and **JSON policies**; enforcing **MFA**; reviewing permissions regularly
- **Customer data:** classifying retail inventory data, encrypting it at rest and in transit, and controlling who can read or export it
- Configuration of every service it uses (for example, making sure a storage bucket or database is not publicly exposed)
- Network controls such as security groups
- For **IaaS** services, the guest operating system, patching, and application security
- Protection of the **root user** of each account

### What transfers and what stays

| Burden | On-Premises | **IaaS** | **PaaS** | **SaaS** |
|---|---|---|---|---|
| Power, cooling, physical security | RetailWave | AWS | AWS | AWS |
| Hardware replacement | RetailWave | AWS | AWS | AWS |
| OS patching | RetailWave | RetailWave | AWS | AWS |
| Application code and configuration | RetailWave | RetailWave | RetailWave | AWS (RetailWave configures) |
| **IAM** (who can access what) | RetailWave | **RetailWave** | **RetailWave** | **RetailWave** |
| **Customer data** protection | RetailWave | **RetailWave** | **RetailWave** | **RetailWave** |

**The two bottom rows never transfer to AWS in any model.** This is the central point for leadership: AWS secures the platform, but a misconfigured **IAM** policy or an exposed data store is RetailWave's responsibility.

### Threat mitigation: preventing unauthorized access to customer inventory data

| Threat | How the proposed IAM structure and shared-responsibility split mitigates it |
|---|---|
| Stolen or guessed credentials | **MFA** is required for the **root user** and for all IAM users (also enforced by an **SCP** at the root of AWS Organizations), so a password alone is not enough. |
| Over-permissioned staff or compromised accounts | Permissions come only from group-attached **JSON policies** following least privilege, which limits the blast radius of any single compromised credential. |
| Developer mistakes reaching production data | **OUs** separate Production from Sandbox into different accounts; Sandbox users have no path to production inventory data. |
| Another AWS tenant accessing RetailWave data | Tenant isolation is part of Security OF the Cloud (AWS responsibility). |
| Physical theft or datacenter failure | Handled by AWS physical controls and by the multi-AZ design. |

---

## 2. Account Foundation & Cost Guardrails: Short-Answer Worksheet

### 2.1 Securing the Root Identity

**The very first action: activate MFA on the root user, before any other administrative action is taken and before any infrastructure is provisioned.**

The **root user** has unrestricted access to everything in the account, including billing and account closure, and its permissions cannot be limited by IAM policies. For this reason, it must be secured first.

**Mandatory process (in this order):**

1. **Sign in as the root user** using the email address used to create the AWS account. Use a shared, company-controlled mailbox (for example, a distribution list), not a personal inbox.
2. **Immediately activate MFA.** Open the account menu, choose **Security credentials**, and select **Assign MFA device**. Use a phishing-resistant method (a passkey or security key) or an authenticator application. **No other administrative action may happen until MFA is active and confirmed.**
3. **Register a second MFA device** and store it separately, so the loss of one device does not lock RetailWave out.
4. **Set a long, unique root password** and store it in the company's approved password vault.
5. **Do not create access keys for the root user.** Delete any that exist.
6. **Create the first administrative identity** (a member of the Admins group in Section 3, with **MFA** required) and use that identity for all daily work.
7. **Lock the root user away.** Use it only for the few tasks that require it (for example, changing the account's support plan or closing the account). Review its sign-in activity regularly.

### 2.2 Implementing Cost Guardrails (AWS Budgets)

**Objective:** Receive a notification as soon as RetailWave's AWS costs reach **$0.01 (USD)**, so unexpected spend is caught from day one.

**Step-by-step configuration of the Zero-Spend template:**

1. Sign in to the AWS Management Console as the administrative identity with billing access. (The first time, the **root user** must enable IAM user and role access to billing information, using **MFA**.)
2. Open **Billing and Cost Management** and choose **Budgets**.
3. Choose **Create budget**.
4. Under budget setup, choose **Use a template (simplified)**.
5. Select the **Zero-Spend template**. It is described as a budget that alerts when spending exceeds the AWS Free Tier limits, with a fixed budgeted amount of **$0.01**.
6. Enter a budget name, for example `RetailWave-Zero-Spend`.
7. Enter the email recipients: the **financial operations team's** distribution list (for example, a shared finops mailbox), plus the VP of Engineering. Using a shared list ensures alerts do not depend on one person.
8. Confirm the alert trigger: **Actual** cost **greater than** the **$0.01** threshold. No percentage buffer or forecast delay is configured, so the alert fires the first time AWS detects spend above **$0.01**.
9. Choose **Create budget**.
10. Verify the budget appears in the Budgets list and that each recipient has confirmed the notification email.

**Financial compliance statement:** the Zero-Spend template notifies the financial operations team when costs reach the **$0.01 USD** threshold, with no manual check required.

**Important limitations to share with leadership (for accuracy):**

- AWS Budgets refreshes its cost data several times per day, so "immediate" means as soon as AWS Budgets detects the charge, not at the exact second it occurs.
- A budget **alerts**; by itself it does not block spending. To turn the alert into an enforced limit, ApexCloud recommends a later phase using **Budget Actions** to attach a deny policy automatically, together with the **SCPs** that already stop Sandbox accounts from launching expensive database instances (see the governance diagram).

---

## 3. IAM Groups & JSON Policies Summary

**IAM** controls *who* can sign in and *what* they can do. RetailWave will define three groups, and every person is added to exactly one group based on their role. Access is never given to individuals directly.

### The three groups

| Group | Members | Access level | Key restrictions |
|---|---|---|---|
| **Admins** | A very small number of trusted engineering leads | Manage IAM, AWS Organizations settings, and billing. Day-to-day account administration. | **MFA** required. Does not use the **root user**. Activity is logged and reviewed. |
| **Developers** | Engineers building and testing the platform | Create and manage compute, storage, and database resources, mainly in the Sandbox account. Limited, read-oriented access in Production. | Cannot change IAM or billing. Sandbox **SCPs** block expensive database instances. No direct access to Production customer data. |
| **Analysts** | Data and business analysts, partner-reporting staff | Read-only access to the specific reports and datasets they need. | Cannot create, modify, or delete infrastructure. Cannot export raw customer inventory data beyond approved datasets. |

### How JSON policies attach to groups

A **JSON policy** is a document that lists which actions are allowed or denied, on which resources, and under what conditions. The approach is:

1. Write one **JSON policy** per role (or use AWS managed policies where they fit).
2. **Attach each JSON policy to the group, not to individual users.**
3. Add users to the matching group. They inherit its permissions.
4. When someone changes role, move them between groups. When they leave, remove them from the group and disable their account. No individual policies need to be tracked down.

Illustrative example, an **Analysts** read-only policy (resource names are placeholders to be finalized in the build phase):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AnalystsReadReportsOnly",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::retailwave-approved-reports",
        "arn:aws:s3:::retailwave-approved-reports/*"
      ]
    },
    {
      "Sid": "DenyWithoutMFA",
      "Effect": "Deny",
      "NotAction": ["iam:ListMFADevices", "iam:EnableMFADevice", "sts:GetSessionToken"],
      "Resource": "*",
      "Condition": { "BoolIfExists": { "aws:MultiFactorAuthPresent": "false" } }
    }
  ]
}
```

### How this enforces the Principle of Least Privilege

- **Start from no access.** IAM denies everything by default; each **JSON policy** grants only the actions a role needs.
- **Role-based, not person-based.** Permissions follow the job, so they are consistent, auditable, and easy to review.
- **Limited blast radius.** If an Analyst's credentials are stolen, the attacker can read approved reports only; they cannot delete data, change IAM, or reach Production infrastructure.
- **Layered guardrails.** **JSON policies** (IAM) define what users can do; **SCPs** (AWS Organizations) define the maximum anything in an account can do. Both must allow an action for it to succeed.
- **Customer data protection.** Production customer inventory data is reachable only through the narrow permissions of the Production accounts, protected by **MFA**, and separated from Sandbox work by **OUs**.

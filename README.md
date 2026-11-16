# RetailWave Analytics: Cloud Architecture and Governance Brief

Prepared by ApexCloud Solutions. This is a design and strategy deliverable; no live AWS infrastructure is provisioned.

## Contents

| File | Purpose |
|---|---|
| `docs/cloud_models_comparison.md` | Maps all 20 On-Premises resources from `on_premises_inventory.csv` to IaaS, PaaS, or SaaS. |
| `docs/infrastructure_and_governance_diagram.md` | Mermaid diagrams: AWS Region with two Availability Zone (AZ) nodes, and the AWS Organizations structure with OUs and SCPs. |
| `docs/security_and_iam_plan.md` | Shared Responsibility summary, root user and Zero-Spend template worksheet, and IAM groups with JSON policies. |

## Reading the diagrams

The diagrams use Mermaid syntax and render automatically in GitHub, GitLab, VS Code (Markdown preview), and most Markdown viewers.

## Key decisions

- **High availability:** AWS Region us-east-2 (Ohio) with Availability Zones us-east-2a and us-east-2b, so the loss of one datacenter does not repeat the Chicago outage.
- **Governance:** Production and Sandbox OUs, with an MFA SCP at the root and a cost-control SCP on Sandbox only.
- **Identity:** MFA on the root user first; Admins, Developers, and Analysts groups with group-attached JSON policies.
- **Cost:** Zero-Spend template in AWS Budgets with a $0.01 USD threshold, notifying the financial operations team.

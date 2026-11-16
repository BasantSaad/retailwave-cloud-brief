# Cloud Models Comparison

**Client:** RetailWave Analytics
**Prepared by:** ApexCloud Solutions
**Source data:** `on_premises_inventory.csv` (20 On-Premises resources)

---

## 1. Service Model Definitions

| Model | What the provider manages | What RetailWave manages | Typical AWS example |
|---|---|---|---|
| **On-Premises** | Nothing. RetailWave owns the building, power, hardware, network, OS, and application. | Everything. | Physical racks in the Chicago data center |
| **IaaS** | Physical data centers, hardware, and virtualization. | Operating system, patching, runtime, scaling configuration, application, and data. | Amazon EC2, Amazon S3, Amazon VPC, Elastic Load Balancing |
| **PaaS** | Everything in IaaS plus the operating system, patching, runtime, and platform scaling. | Application code, configuration, and data. | Amazon RDS, AWS Lambda, Amazon SQS, AWS Elastic Beanstalk |
| **SaaS** | The entire stack, including the application itself. | User access (IAM), configuration, and data. | Amazon QuickSight, Amazon SES, IAM Identity Center |

**Rule used for assignment:** the model is chosen by the *lowest level of the stack RetailWave still needs to control*. If the pain point is hardware only, **IaaS** is enough. If it involves OS patching, scaling, or failover, **PaaS** removes the burden. If RetailWave only needs to consume a finished tool, **SaaS** applies.

---

## 2. Resource-by-Resource Comparison

| # | Resource | Current On-Premises Setup | Pain Point | Assigned Model | Target AWS Service | Why this model |
|---|---|---|---|---|---|---|
| 1 | Inventory API Gateway | Physical rack, Chicago DC, Zone A | Hardware bottleneck at peak retail seasons | **PaaS** | Amazon API Gateway | Throughput scales automatically; no hardware or OS to manage. |
| 2 | Primary Inventory DB | Chicago DC SAN storage | Single point of failure during the 48-hour outage | **PaaS** | Amazon RDS (Multi-AZ) | Managed database with a standby in a second Availability Zone (AZ). |
| 3 | Replica Inventory DB | Chicago DC SAN storage | Replication lag and manual failover | **PaaS** | Amazon RDS (Multi-AZ) | Replication and failover are handled by the platform. |
| 4 | Historical Data Warehouse | Legacy physical server rack | Costly hardware refresh every 3 years | **PaaS** | Amazon Redshift | No hardware lifecycle; the provider manages the platform. |
| 5 | Real-Time Analytics Engine | Chicago DC blade server (Row 4) | CPU maxes out on Black Friday, no auto-scaling | **IaaS** | Amazon EC2 with Auto Scaling | RetailWave keeps control of its analytics runtime while capacity scales with demand. |
| 6 | ETL Processing Nodes | On-premise VM cluster | Cannot scale down off-peak; wasted power and cooling | **PaaS** | AWS Glue | Pay per job run; no idle cluster. |
| 7 | Flat File Archive (CSV/JSON) | Local Network Attached Storage (NAS) | Capacity limits, manual drive swaps | **IaaS** | Amazon S3 | Virtually unlimited storage capacity, billed by usage. |
| 8 | Hardware Load Balancer | Physical appliance, Chicago DC | Geographic routing impossible without a second site | **IaaS** | Elastic Load Balancing | Virtual network infrastructure spanning multiple AZs. |
| 9 | Message Queue Server | Bare metal Linux server | OS patching forces downtime | **PaaS** | Amazon SQS | No server or OS to patch. |
| 10 | Internal BI Dashboard | Dedicated VM running BI software | Licensing and maintenance overhead | **SaaS** | Amazon QuickSight | Finished application; RetailWave only manages users and data. |
| 11 | Employee Directory | Local Active Directory server | Hard to apply zero-trust access for remote engineers | **SaaS** | AWS IAM Identity Center | Hosted identity service with centralized access policies. |
| 12 | Nightly Backup Storage | Tape drives in Chicago | 48-hour restore time, degrading tapes | **IaaS** | Amazon S3 (cross-AZ durable storage) with AWS Backup | Durable cloud storage replaces physical media. |
| 13 | ML Model Training Node | Local GPU workstation | No strict access controls on sensitive retail data | **IaaS** | Amazon EC2 GPU instances governed by IAM | RetailWave controls the training environment; access is locked down with IAM and JSON policies. |
| 14 | Partner Web Portal | Windows Server 2012 rack | End-of-life OS and compliance risk | **PaaS** | AWS Elastic Beanstalk | The platform supplies a supported, patched OS. |
| 15 | Perimeter Firewall | Physical firewall appliance | Configuration drift; cannot block distributed attacks dynamically | **IaaS** | Amazon VPC security groups and network ACLs | Virtual network controls defined as policy, applied consistently. |
| 16 | CI/CD Build Server | Dedicated machine in server closet | Overheats; build queue blocks deployments | **PaaS** | AWS CodeBuild | Managed, on-demand build capacity. |
| 17 | Log Aggregation Server | Local bare metal server | Storage fills; logs dropped | **PaaS** | Amazon CloudWatch Logs | Managed ingestion and retention. |
| 18 | Email Notification Server | On-premise SMTP server | IP reputation causes delayed partner alerts | **SaaS** | Amazon SES | Finished email service with managed sending reputation. |
| 19 | Partner API Sandbox | Partitioned local VM | Sandbox contention affects production | **IaaS** | Amazon EC2 in a separate Sandbox AWS account | Isolation via separate accounts (see the governance diagram). |
| 20 | Data Ingestion Webhooks | Node.js app on one physical server | Spiky traffic drops JSON payloads | **PaaS** | AWS Lambda behind Amazon API Gateway | Event-driven scaling absorbs spikes. |

---

## 3. Summary by Model

| Model | Count | Resources (by #) |
|---|---|---|
| **On-Premises** (current state) | 20 | All |
| **IaaS** | 7 | 5, 7, 8, 12, 13, 15, 19 |
| **PaaS** | 10 | 1, 2, 3, 4, 6, 9, 14, 16, 17, 20 |
| **SaaS** | 3 | 10, 11, 18 |

**Key takeaway for leadership:** half of the estate moves to **PaaS**, which removes the OS patching, hardware refresh, and manual failover burdens behind most of the pain points. **IaaS** is used only where RetailWave needs deeper control. **SaaS** covers commodity tools that RetailWave should not run itself.

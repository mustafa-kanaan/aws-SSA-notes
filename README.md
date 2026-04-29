# 📚 AWS Solutions Architect Associate (SAA-C03) — Study Notes

> Personal study notes for the AWS SAA-C03 certification exam.
> Each section includes concepts, ASCII diagrams, comparison tables, and exam traps.

---

## 🎯 Exam Info

> ⚠️ **Always verify the latest exam details on the [official AWS certification page](https://aws.amazon.com/certification/certified-solutions-architect-associate/) before booking.**

| Detail | Value |
|---|---|
| Exam Code | SAA-C03 |
| Duration | 130 min (160 min with ESL accommodation) |
| Questions | 65 scored + 15 unscored |
| Passing Score | 720 / 1000 |
| Format | Multiple choice & multiple response |
| Delivery | Online proctored or test center |

### Exam Domains

| Domain | Weight |
|---|---|
| Design Secure Architectures | 30% |
| Design Resilient Architectures | 26% |
| Design High-Performing Architectures | 24% |
| Design Cost-Optimized Architectures | 20% |

---

## 📖 Table of Contents

| # | Section | Key Topics | Status |
|---|---|---|---|
| 01 | [IAM & VPC Fundamentals](./01-IAM-and-VPC/notes.md) | IAM users/roles/policies, VPC, Subnets, IGW, NAT, Bastion Host, Hybrid Connectivity | ✅ Done |
| 02 | [Key Architecture Pillars](./02-Architecture-Pillars/notes.md) | Elasticity, HA, Reliability, CloudWatch, DR, Microservices | ✅ Done |
| 03 | [VPC Deep Dive](./03-VPC-Deep-Dive/notes.md) | NAT, Bastion, Peering, Transit Gateway, VPC Endpoints | ✅ Done |
| 04 | [Hybrid Cloud Networking](./04-Hybrid-Networking/notes.md) | VPN, Direct Connect, Transit Gateway | ✅ Done |
| 05 | [Compute — EC2 & EBS](./05-EC2-and-EBS/notes.md) | Instance types, EBS volumes, AMI, placement groups | 🔄 In Progress |
| 06 | [Scaling & Load Balancing](./06-ELB-and-Auto-Scaling/notes.md) | ALB, NLB, GWLB, Auto Scaling groups, policies | ⏳ Pending |
| 07 | [Relational Databases — RDS](./07-RDS/notes.md) | Multi-AZ, Read Replicas, Aurora, RDS Proxy, backups | ⏳ Pending |
| 08 | [NoSQL Databases](./08-NoSQL/notes.md) | DynamoDB, ElastiCache, DocumentDB, Redshift | ⏳ Pending |
| 09 | [IAM Intermediate](./09-IAM-Intermediate/notes.md) | Resource policies, SCP, permission boundaries, federation | ⏳ Pending |
| 10 | [S3 Deep Dive](./10-S3/notes.md) | Storage classes, lifecycle, encryption, versioning, replication, Object Lock | ⏳ Pending |
| 11 | [Route 53 & CloudFront](./11-Route53-CloudFront/notes.md) | Routing policies, Global Accelerator, CDN, OAC | ⏳ Pending |
| 12 | [Serverless Computing](./12-Serverless/notes.md) | Lambda, API Gateway, SAM | ⏳ Pending |
| 13 | [Storage Services](./13-Storage-Services/notes.md) | EFS, FSx, Storage Gateway, Snow Family, DataSync, Backup | ⏳ Pending |
| 14 | [Containers](./14-Containers/notes.md) | ECS, EKS, ECR, App Runner, Fargate | ⏳ Pending |
| 15 | [Messaging & Integration](./15-Messaging/notes.md) | SQS, SNS, EventBridge, MQ, Step Functions | ⏳ Pending |
| 16 | [Monitoring & Auditing](./16-Monitoring/notes.md) | CloudWatch, CloudTrail, X-Ray, Config | ⏳ Pending |
| 17 | [Governance & Operations](./17-Governance/notes.md) | AWS Organizations, CloudFormation, SSM, Control Tower | ⏳ Pending |
| 18 | [Security, Identity & Compliance](./18-Security/notes.md) | KMS, CloudHSM, Shield, WAF, GuardDuty, Macie, Inspector, Cognito | ⏳ Pending |
| 19 | [Analytics](./19-Analytics/notes.md) | Athena, Glue, Kinesis, EMR, QuickSight, Redshift Spectrum | ⏳ Pending |
| 20 | [Well-Architected Framework](./20-Well-Architected/notes.md) | 6 pillars: Operational Excellence, Security, Reliability, Performance, Cost, Sustainability | ⏳ Pending |

---

## 🗂️ Status Legend

| Icon | Meaning |
|---|---|
| ✅ Done | Notes complete and reviewed |
| 🔄 In Progress | Currently studying |
| ⏳ Pending | Not started yet |
| 🔁 Revisit | Done but needs review — identified gaps |

---

## 📝 Notes Format

Each section's `notes.md` follows a consistent structure:

```
# Section Title
## Concept 1
  - Explanation
  - ASCII diagrams
  - Comparison tables
  - ⚠️ Exam traps
  - ✅ Best practices
## Quick Reference — Exam Traps Summary
```

---

## 🧠 Study Approach

### Per Section
1. Study the section material
2. Write notes in your own words — do not copy/paste
3. Draw the architecture from memory before looking at the diagram
4. Review the **Exam Traps** table at the end of each notes file
5. Update status in this README

### Mid-Point Review (After Section 10)
- Do a full mock exam
- Any domain scoring below 75% → revisit relevant sections
- Mark those sections as 🔁 Revisit

### Final Preparation
- Do 2–3 full-length mock exams under timed conditions
- Review all ⚠️ exam trap callouts across all notes
- Focus revision time on your weakest domain

### Exam Day
- Read every question fully before selecting an answer
- Eliminate clearly wrong options first
- Watch for keywords: *most cost-effective*, *least operational overhead*, *highly available*, *fault-tolerant*

---

## 🔗 Useful Resources

| Resource | Link |
|---|---|
| Official Exam Guide | [SAA-C03 Exam Guide](https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf) |
| AWS Well-Architected Framework | [AWS Docs](https://aws.amazon.com/architecture/well-architected/) |
| AWS Whitepapers | [AWS Whitepapers](https://aws.amazon.com/whitepapers/) |
| AWS Free Tier (for hands-on practice) | [AWS Free Tier](https://aws.amazon.com/free/) |
| Official Practice Question Set | [Skill Builder](https://explore.skillbuilder.aws/) |

---

*Started: April 2026*

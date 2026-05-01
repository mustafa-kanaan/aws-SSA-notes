# AWS SAA — PDF 1: Cloud Foundations & AWS Introduction

> 📌 **Study tip:** Read each section, then close the doc and try to recall the key points. Use the "Exam Traps" boxes to review before your test.

---

## 1. Cloud Computing Fundamentals

### What is Cloud Computing?
Cloud computing is the **on-demand delivery** of IT resources over the internet with **pay-as-you-go pricing**.
- A **data center** alone is NOT a cloud — it needs orchestration/automation layers to become a cloud.

### Cloud Deployment Models

| Model | Description | Best For |
|---|---|---|
| **Public Cloud** | Shared, multi-tenant (e.g. AWS) | Less confidential, cost-effective |
| **Private Cloud** | Single-tenant, dedicated | Confidential data, regulatory |
| **Hybrid Cloud** | Public + private, orchestrated | DR, backup, dev/test |
| **Multi-Cloud** | Multiple public providers | Avoid vendor lock-in |

### Cloud Service Models

| Layer | SaaS | PaaS | IaaS | On-Premises |
|---|---|---|---|---|
| App | Provider | Provider | Customer | Customer |
| OS | Provider | Provider | Customer | Customer |
| Hardware | Provider | Provider | Provider | Customer |

---

## 2. AWS Global Infrastructure

- **30+ Regions**, **96+ AZs**, **410+ Points of Presence**
- Each **Region** = separate geographic area with multiple AZs
- Each **AZ** = one or more data centers with independent power, cooling, networking
- AZs within a region are connected via **low-latency private fiber**

### How to Choose a Region
1. Data compliance & governance
2. Latency — closest to your users
3. Service availability — not all services in all regions
4. Pricing — differs per region

---

## 3. IAM 101

### IAM Identities

| Identity | Used For | Credential Type |
|---|---|---|
| **IAM User** | Human users | Username + password or Access Keys |
| **IAM Group** | Organize users | Inherits from policies |
| **IAM Role** | Services, apps, federated users | Temporary via STS |

```
IAM User  →  Permanent credentials  →  Long-term risk if leaked
IAM Role  →  Temporary credentials (STS, auto-expire 1–12h)  →  Safer
```

### IAM Best Practices
- 🔒 Lock root credentials — use only for account-level tasks
- 🔑 Enable MFA for root and all privileged users
- 📉 Grant **least privilege**
- 🔄 Use **roles** instead of sharing credentials
- 🗑️ Remove unused credentials, rotate access keys
- 📋 Monitor via **CloudTrail**

### Root-Only Tasks
- Change account name/email/root password
- Close the AWS account
- Change/cancel AWS Support plan

> ⚠️ **Never hardcode access keys** in code or EC2. Use IAM Roles.

---

## 4. VPC 101

```
AWS Region
┌─────────────────────────────────────────────┐
│  Your VPC  (10.0.0.0/16)                    │
│  AZ-a                    AZ-b               │
│  ┌─────────────┐         ┌─────────────┐    │
│  │Public Subnet│         │Public Subnet│    │
│  │10.0.10.0/24 │         │10.0.20.0/24 │    │
│  └─────────────┘         └─────────────┘    │
│  ┌─────────────┐         ┌─────────────┐    │
│  │Private Sub  │         │Private Sub  │    │
│  │10.0.100.0/24│         │10.0.200.0/24│    │
│  └─────────────┘         └─────────────┘    │
└─────────────────────────────────────────────┘
```

**Key rules:**
- Main CIDR **cannot be changed** after creation (add up to 4 secondary CIDRs)
- Subnets **cannot overlap** and **cannot span AZs**
- A subnet is public **only when BOTH** are true:
  - VPC has an IGW attached
  - Subnet route table has `0.0.0.0/0 → IGW`

### IP Address Types

| Type | Internet Routable | Released on Stop? | Cost |
|---|---|---|---|
| **Private IP** | ❌ No | No | Free |
| **Public IP** | ✅ Yes | Yes | Free |
| **Elastic IP** | ✅ Yes | No | Free if in use |

---

## 5. Security Groups & NACLs

| Feature | Security Group | NACL |
|---|---|---|
| Level | Instance (ENI) | Subnet |
| Rules | Allow only | Allow & Deny |
| State | **Stateful** | **Stateless** |
| Rule eval | All at once | Ordered (lowest # first) |

**Stateless NACL trap:**
```
Allow inbound port 80  →  Must ALSO allow outbound 1024-65535
Otherwise response is dropped ❌
```

> 💡 To **block a specific IP** → use a **NACL DENY rule**. Security Groups cannot deny.

---

## 6. Hybrid Connectivity 101

| | **VPN** | **Direct Connect (DX)** |
|---|---|---|
| Path | Public internet (IPSec) | Private dedicated fiber |
| Deploy time | Hours | Weeks to months |
| Latency | Inconsistent | Low and consistent |
| Encryption | ✅ Yes | ❌ No (by default) |
| Cost | Low | High |

> ⚠️ DX is **private but NOT encrypted**. For encryption + low latency → **VPN tunnel over DX**.

---

## 7. EC2 101

### Instance Type Categories

| Category | Use Case |
|---|---|
| General Purpose (t3, m5) | Balanced compute/memory/networking |
| Compute Optimized (c5) | High CPU — HPC, gaming |
| Memory Optimized (r5) | In-memory DBs, large caches |
| Storage Optimized (i3) | High I/O, large local datasets |
| Accelerated (p3, g4) | ML, GPU workloads |

### Pricing Models

| Model | Best For | Savings |
|---|---|---|
| On-Demand | Short-term, unpredictable | — |
| Reserved (1 or 3 yr) | Steady-state workloads | Up to 72% |
| Spot | Fault-tolerant, flexible | Up to 90% |
| Dedicated Host | License/regulatory compliance | — |

> ⚠️ Spot instances can be **interrupted** with 2-min notice.

### Accessing Private EC2

| Method | Port 22 Needed? | Best For |
|---|---|---|
| Bastion Host | Yes | Traditional |
| EC2 Instance Connect | Yes (internally) | Quick/one-off |
| SSM Session Manager | **No** | ✅ Production best practice |

---

## 8. EBS 101

- **Persistent block storage** for EC2 — survives stop/start
- **AZ-locked** — must be in same AZ as EC2
- To move to another AZ: **snapshot → restore in new AZ**
- Encrypted via KMS at rest

### Volume Types

| Type | Use Case |
|---|---|
| gp3/gp2 (SSD) | General purpose, boot volumes |
| io2/io1 (SSD) | High-performance DBs |
| st1 (HDD) | Throughput-intensive (big data, logs) |
| sc1 (HDD) | Cold, infrequently accessed |

---

## 9. Encryption 101

| Type | Protects | When |
|---|---|---|
| **At Rest** | Stored data (disk, S3, DB) | Data sitting still |
| **In Transit** | Data over network | Data traveling (SSL/TLS) |

### Symmetric vs Asymmetric

| | Symmetric | Asymmetric |
|---|---|---|
| Keys | One shared key | Key pair (public + private) |
| Speed | Fast | Slow |
| Use case | Bulk data encryption | Key exchange, digital signatures |

> 💡 HTTPS uses **asymmetric** to exchange a key, then **symmetric** to encrypt actual data.

---

## 10. KMS 101

- Managed service for creating and controlling **encryption keys**
- KMS keys **never leave KMS unencrypted**
- All key usage logged in **CloudTrail**
- Supports automatic **annual key rotation**
- Integrated with S3, EBS, RDS, Secrets Manager, etc.

> ⚠️ Lose access to the KMS key → **cannot decrypt data**, even as account owner.

---

## 11. SNS 101

**Pub/Sub service** — one message, many receivers (push-based).

```
SNS TOPIC
    │
    ├──► 📧 Email
    ├──► 📱 SMS
    ├──► 🔁 SQS Queue
    └──► ⚡ Lambda
```

- Messages are **NOT stored** — fire and forget
- Use case: fan-out notifications to multiple systems simultaneously

---

## 12. SQS 101

**Message queue** — one message, one receiver (poll-based).

```
Producer ──► [msg1][msg2][msg3] QUEUE ──► Consumer (polls)
```

- Messages **persist** up to 14 days
- Consumer polls → processes → **deletes** message
- Use case: **decouple** producer from consumer

### SNS vs SQS

| | SNS | SQS |
|---|---|---|
| Pattern | Pub/Sub (push) | Queue (poll) |
| Receivers | Many simultaneously | One at a time |
| Storage | ❌ No | ✅ Yes (14 days) |
| Use case | Fan-out | Decouple workloads |

> 💡 **Fan-out pattern:** SNS → multiple SQS queues, each processed independently.

---

## 13. CloudFront 101

AWS's **CDN** — caches content at **400+ Edge Locations** close to users.

```
User → Edge Location (cache) → Origin (S3 / ALB / EC2)
         ↑ serves from cache      ↑ only on cache miss
```

- Reduces latency by serving from nearest edge
- Supports static (S3) and dynamic (ALB/EC2) origins
- Built-in **DDoS protection** (Shield Standard — free)

---

## 14. Route 53 101

AWS's **managed DNS service** — 3 functions:

1. **Register domain names** → buy yoursite.com
2. **DNS service** → map name to IP
3. **Health checks** → monitor resource availability

### Hosted Zones

| Zone | Resolves For |
|---|---|
| **Public** | Internet-facing traffic |
| **Private** | Traffic inside your VPC only |

---

## 15. RDS 101

**Fully managed relational DB** — AWS handles patching, backups, replication, scaling.

**Supported engines:** MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora

```
✅ Lives inside your VPC (private subnets best practice)
✅ Security Groups attachable
✅ Encryption at rest (KMS) and in transit (SSL/TLS)
❌ No OS/root access
```

### Deployment Modes

| Mode | Description |
|---|---|
| Standalone | Single instance — dev/test |
| Multi-AZ | Primary + standby + auto failover — production HA |
| Read Replicas | Primary + read replicas — read-heavy workloads |

### OLTP vs OLAP

| | OLTP | OLAP |
|---|---|---|
| Purpose | Day-to-day transactions | Analytics & reporting |
| Query type | Simple, fast | Complex, slow |
| Data | Current | Historical |
| AWS Service | **RDS / Aurora** | **Amazon Redshift** |
| Examples | Orders, logins, ATM | Sales reports, revenue forecasts |

> ⚠️ "data warehouse / analytics / BI" → **Redshift**. "application DB / transactions" → **RDS**.

---

## 16. AWS Market Advantages (6 Cloud Advantages)

| # | Advantage | Meaning |
|---|---|---|
| 1 | Trade CAPEX for OPEX | No upfront hardware — pay as you use |
| 2 | Massive Economies of Scale | AWS scale → lower costs for you |
| 3 | Stop Guessing Capacity | Scale on demand |
| 4 | Increased Speed & Agility | Deploy in minutes not months |
| 5 | Stop Spending on Data Centers | Focus on app not infrastructure |
| 6 | Go Global in Minutes | Deploy to any region instantly |

---

## Exam Traps Quick Reference

| Trap | Correct Answer |
|---|---|
| Subnet private even with IGW | Route table must ALSO have `0.0.0.0/0 → IGW` |
| NACL inbound ok but traffic drops | Add outbound ephemeral ports 1024–65535 |
| Block a specific IP | NACL DENY rule (SG can't deny) |
| Only one per VPC | IGW, VGW |
| Direct Connect = encrypted? | Private yes, **NOT encrypted** — add VPN for encryption |
| Access private EC2 no port 22 | SSM Session Manager |
| Hardcoded access keys leaked | Use IAM Roles (temporary credentials) |
| Analytics/reporting DB | Redshift (OLAP), not RDS |
| Move EBS to another AZ | Snapshot → restore in new AZ |
| SNS vs SQS receivers | SNS = many (fan-out), SQS = one at a time |

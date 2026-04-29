# AWS SAA — Section 1: IAM & VPC Fundamentals

> 📌 **Study tip:** Read each section, then close the doc and try to recall the key points. Use the "Exam Traps" boxes to review before your test.

---

## 1. AWS Identity & Access Management (IAM)

### What is IAM?

IAM is AWS's global identity and access control service. It answers three questions for every request made to AWS:

- **Who are you?** → Authentication
- **What are you allowed to do?** → Authorization
- **Under what conditions?** → Policy conditions

IAM is **global** — it is not tied to any specific AWS region.

---

### IAM Identities

| Identity | Used For | Credential Type |
|---|---|---|
| **IAM User** | Human users (console/CLI access) | Username + password, or Access Keys |
| **IAM Group** | Organizing users with shared permissions | N/A — inherits from attached policies |
| **IAM Role** | AWS services, applications, federated users | Temporary credentials via STS |

#### Key Distinction: User vs. Role

```
IAM User  →  Permanent credentials  →  Long-term risk if leaked
IAM Role  →  Temporary credentials (STS, auto-expire in 1–12h)  →  Safer
```

**Always use roles for:**
- EC2 instances that need access to S3, DynamoDB, etc.
- CI/CD pipelines (CodeBuild, GitHub Actions OIDC)
- Cross-account access
- Federated users (company SSO/Active Directory)

> ⚠️ **Never hardcode access keys** in code or on EC2 instances. If the instance is compromised, the attacker gets permanent credentials usable from anywhere in the world.

---

### IAM Best Practices

- 🔒 Lock away root account credentials — use only for account-level tasks
- 🔑 Enable MFA for root and all privileged users
- 📉 Grant **least privilege** — only the permissions needed, nothing more
- 🔄 Use roles instead of sharing credentials between services
- 🗑️ Remove unused credentials and rotate access keys regularly
- 📋 Monitor activity via CloudTrail

#### Why Least Privilege Matters

If an account with `AdministratorAccess` is compromised:
- Attacker can delete all resources
- Attacker can exfiltrate all data
- Attacker can modify billing or close the account

With least privilege, the blast radius is limited to only what that identity could access.

---

### Tasks That Require Root Credentials

Some actions can **only** be done by the root user:
- Change account name, email, or root password
- Close the AWS account
- Change or cancel AWS Support plan
- Register as a seller in the Reserved Instances Marketplace

---

## 2. AWS Virtual Private Cloud (VPC)

### What is a VPC?

A VPC is your **isolated virtual network** inside AWS — think of it as your own private data center in the cloud. By default, VPCs are completely isolated from each other.

```
AWS Region (e.g., us-east-1)
┌─────────────────────────────────────────────┐
│  Your VPC  (10.0.0.0/16)                    │
│                                             │
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
- One VPC per region (spans all AZs in that region)
- Main CIDR block **cannot be changed** after creation (add up to 4 secondary CIDRs)
- Subnets **cannot overlap** and **cannot span AZs** (one subnet = one AZ)
- Intra-VPC routing is **automatic** — all subnets can communicate by default via the implied router

---

### VPC Core Components

#### Implied Router & Route Tables

The implied router handles all traffic between subnets and to the outside world. You cannot access it directly — AWS manages it.

Every VPC has a **main route table** by default. Rules:
- A subnet can be associated with **only one** route table
- One route table can serve **multiple** subnets
- The **local route** (`10.0.0.0/16 → local`) is always present and cannot be deleted

> **Why custom route tables?** If you add `0.0.0.0/0 → IGW` to the main route table, ALL subnets become public — including your database. Custom route tables let you control which subnets are public and which are private.

---

#### Internet Gateway (IGW)

The IGW is the door between your VPC and the public internet.

- Fully managed, highly available, never a bottleneck
- **Only one IGW per VPC**
- Supports IPv4 and IPv6

**A subnet is public ONLY when BOTH conditions are met:**

```
Condition 1: VPC has an IGW attached
Condition 2: Subnet's route table has  0.0.0.0/0 → IGW
```

If either is missing, the subnet is private — even if an EC2 instance has a public IP.

---

#### NAT Gateway

Allows **private instances** to initiate outbound internet connections (e.g., OS patches, pulling Docker images) without being reachable from the internet.

```
Private EC2
    │
    ▼
NAT Gateway  (lives in public subnet, has Elastic IP)
    │
    ▼
IGW → Internet
```

- Managed by AWS — no maintenance needed
- **Highly available within one AZ only**
- For full HA: deploy **one NAT Gateway per AZ**, each with its own route table

> ⚠️ **Exam trap:** One NAT Gateway in AZ-a goes down → private instances in AZ-b lose outbound internet too, because they were routing through it. Fix: one NAT GW per AZ.

---

### IP Address Types in AWS

| Type | Internet Routable | Released on Stop? | Changes on Stop? | Cost |
|---|---|---|---|---|
| **Private IP** | ❌ No | No | No | Free |
| **Public IP** | ✅ Yes | Yes | Yes | Free |
| **Elastic IP (EIP)** | ✅ Yes | No | No | Free if in use |

**Elastic IP use case:** Whitelisting — give a 3rd-party service your EIP so they can whitelist it in their firewall. Your IP never changes even if the instance is replaced.

---

### Security: Two Layers of Defense

#### Security Groups vs. Network ACLs (NACLs)

| Feature | Security Group | NACL |
|---|---|---|
| Applies at | Instance level (ENI) | Subnet level |
| Rule types | Allow only | Allow **and** Deny |
| State | **Stateful** | **Stateless** |
| Rule evaluation | All rules evaluated | Processed in order (lowest number first) |
| Scope | Only attached instances | All instances in the subnet |
| Default | Deny all inbound, allow all outbound | Allow all (default NACL) |

#### Stateful vs Stateless — The Critical Difference

**Security Group (Stateful):**
```
Allow inbound HTTP port 80  →  Return traffic automatically allowed ✅
No outbound rule needed for the response
```

**NACL (Stateless):**
```
Allow inbound HTTP port 80  →  Must ALSO allow outbound 1024-65535 ✅
Otherwise the response is dropped at the subnet boundary ❌
```

The ephemeral port range (1024–65535) is used by the **client OS**, chosen randomly. You cannot predict the exact port, so you must allow the full range in your NACL outbound rules.

> ⚠️ **Most common exam trap:** Traffic works fine with security groups but breaks when NACLs are added. Answer is always the missing ephemeral port outbound rule.

---

### Accessing Private Instances

#### Bastion Host (Jump Server)

A Bastion Host is an EC2 instance in a **public subnet** used as the secure SSH entry point to reach private instances.

```
Your PC  →(SSH port 22)→  Bastion (public subnet)  →(SSH port 22)→  Private EC2
```

**Security group rules:**
- Bastion inbound: allow SSH from **your IP only** (`your-ip/32`)
- Bastion outbound: allow SSH to **private EC2's security group only**
- Private EC2 inbound: allow SSH from **Bastion's security group ID**

> 💡 Use **SSH agent forwarding** (`ssh -A`) so your private key never leaves your PC and is never stored on the Bastion.

#### Three Methods to Access Private EC2

| Method | How | Port 22 Needed? | Best For |
|---|---|---|---|
| **Bastion Host** | SSH through jump server | Yes | Traditional access |
| **EC2 Instance Connect** | AWS console browser | Yes (internally) | Quick/one-off access |
| **SSM Session Manager** | AWS Systems Manager | **No** | Production best practice |

> ✅ **Modern best practice:** SSM Session Manager — no open ports, fully audited, works with no public IP.

---

## 3. Hybrid Connectivity

Connecting your on-premises data center to AWS.

### Virtual Private Gateway (VGW)

The VGW is the **AWS-side termination point** for hybrid connections. It attaches to your VPC. Like the IGW, only **one VGW per VPC** at a time.

---

### VPN vs Direct Connect

```
On-premises                              AWS VPC
Data Center                              
    │                                       │
[Customer Gateway]──VPN (internet)──[VGW]──┤
    │                                       │
[Customer Gateway]──DX (private fiber)─────┘
```

| | **VPN** | **Direct Connect (DX)** |
|---|---|---|
| Network path | Public internet (encrypted IPSec) | Private dedicated fiber |
| Deploy time | Hours | Weeks to months |
| Reliability | Variable (internet-dependent) | High (dedicated line) |
| Latency | Inconsistent | Low and consistent |
| Cost | Low | High |
| Encryption | ✅ Yes (IPSec) | ❌ No (by default) |
| Use case | Quick setup, backup path | Production, large data transfers |

> ⚠️ **Critical exam trap:** Direct Connect is **private but NOT encrypted**. If you need low latency AND encryption, run a **VPN tunnel over Direct Connect**.

---

### Recommended Pattern: DX Primary + VPN Backup

```
Normal:      On-prem ──DX (preferred)──► VGW ──► VPC
DX failure:  On-prem ──VPN (backup)───► VGW ──► VPC  ← automatic via BGP
```

BGP (Border Gateway Protocol) handles automatic failover:
- Both connections advertise the same routes
- DX is preferred (higher BGP priority)
- If DX fails, BGP automatically switches to VPN
- When DX recovers, traffic shifts back automatically

> ✅ **Best practice for most production workloads:** DX as primary + VPN as backup. Cost-effective because the VPN is only used during DX failure.

---

## Quick Reference: Exam Traps Summary

| Trap | Correct Answer |
|---|---|
| Subnet is still private even with IGW attached | Route table must ALSO point `0.0.0.0/0 → IGW` |
| NACL allows inbound but traffic drops | Must also allow outbound ephemeral ports 1024–65535 |
| Only one of these per VPC | IGW, VGW |
| NAT GW fails for other AZ instances | Need one NAT GW **per AZ** |
| Direct Connect is secure? | Private yes, but **not encrypted** — add VPN for encryption |
| Access private EC2 without port 22 open | SSM Session Manager |
| Hardcoded access keys leaked | Use IAM roles instead — temporary credentials only |

---

*AWS Solutions Architect Associate — DolfinED Course | Section 1 Notes*

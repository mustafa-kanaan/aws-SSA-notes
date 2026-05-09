# AWS VPC Deep Dive — NAT, Bastion, Peering, Transit Gateway, Endpoints, Flow Logs & Local Zones

## Overview

This document covers the AWS Virtual Private Cloud (VPC) deep dive topics: NAT Instance vs NAT Gateway, Bastion Hosts, VPC Peering, Transit Gateway, VPC Endpoints, IPv6 Egress-Only Gateway, VPC Flow Logs, and AWS Local Zones. These are foundational networking concepts for building secure, scalable, and highly available cloud architectures.

---

## NAT — Network Address Translation

### Why NAT?

Instances in private subnets have no public IP and cannot reach the internet directly. NAT allows outbound internet traffic from private subnets while blocking inbound connections initiated from the internet.

### NAT Instance

A NAT Instance is a regular EC2 instance manually configured to perform NAT. It must be launched in a **public subnet**, and the private subnet route table must have a default route pointing to the NAT instance ID.

> **Important:** You must **disable Source/Destination Check** on the NAT instance. By default, EC2 drops packets where it is not the source or destination — disabling this allows the instance to act as a network intermediary.

### NAT Gateway

A fully managed, highly available AWS service that performs NAT at scale.

- Scales up to **45 Gbps** per gateway
- Cannot be associated with a security group
- For HA: deploy **one NAT Gateway per AZ**, and point each AZ's private route table to the NAT Gateway in the same AZ

```
AZ-A: Private Subnet → NAT GW (AZ-A) → IGW → Internet
AZ-B: Private Subnet → NAT GW (AZ-B) → IGW → Internet
```

**Public NAT Gateway** — requires an Elastic IP, used for internet-bound traffic.

**Private NAT Gateway** — no Elastic IP, used to route traffic to other VPCs or on-premises through a Transit Gateway or VGW. Traffic from a private NAT Gateway is dropped by the VPC's IGW.

### NAT Instance vs NAT Gateway

| Feature | NAT Gateway | NAT Instance |
|---|---|---|
| **Availability** | Highly available per AZ | Requires failover script |
| **Performance** | Up to 45 Gbps | Depends on instance type |
| **Management** | Fully managed by AWS | Customer managed |
| **Elastic IP** | Elastic IP only (public) | Elastic or Public IP |
| **Security Groups** | Cannot associate | Can associate |
| **SSH / Bastion use** | No | Yes |
| **Pricing** | $0.045/hr + $0.045/GB | EC2 instance pricing |

**Cost tip:** Launch NAT Gateway in the same AZ as your private instances — intra-AZ traffic between EC2 and NAT Gateway is not charged.

---

## Bastion Host (Jump Server)

A Bastion Host is a hardened EC2 instance in a **public subnet** used to securely SSH (Linux) or RDP (Windows) into private instances.

### Security Best Practices

- Use **Elastic IPs** (not public IPs) for the Bastion — stable, predictable address
- Restrict source IPs in **Security Groups and NACLs** to the corporate data center CIDR only
- Port **22** for Linux Bastion, port **3389** for Windows Bastion
- Deploy in an **Auto Scaling Group** for HA

```
Corporate Network → Bastion (Public Subnet) → Private EC2 / RDS (Private Subnet)
     Port 22/3389         Port 22/3389
```

### Bastion vs NAT Gateway

| Purpose | Service |
|---|---|
| Outbound internet for private instances | NAT Gateway |
| Admin SSH/RDP access to private instances | Bastion Host |

> **From our discussion:** Developers who need to connect DB clients (e.g., MySQL Workbench, Toad) to RDS in private subnets can use a Bastion with **SSH port forwarding** — tunneling a local port through the Bastion to the RDS endpoint. This avoids needing to expose RDS publicly.

---

## VPC Peering

VPC Peering creates a direct private network route between two VPCs — same account, different accounts, or different regions.

### Rules

- The two VPCs **cannot have overlapping CIDR ranges**
- You cannot create a second peering connection between the same pair of VPCs
- **Transitive peering is not supported** — if VPC-A peers with VPC-B, and VPC-B peers with VPC-C, VPC-A cannot reach VPC-C through VPC-B

```
VPC-A ──── VPC-B ──── VPC-C

❌ VPC-A cannot reach VPC-C via VPC-B (no transitive routing)
✅ VPC-A must peer directly with VPC-C
```

- **Edge-to-edge routing is not supported** — on-premises cannot use a VPC Peering connection to reach another VPC's resources

### Scaling Problem

With N VPCs, full mesh requires **N(N-1)/2** peering connections. This becomes unmanageable at scale — Transit Gateway solves this.

---

## AWS Transit Gateway (TGW)

Transit Gateway is a **regional** network hub that connects VPCs, VPNs, and Direct Connect in a hub-and-spoke model — replacing complex mesh peering.

```
Without TGW:                    With TGW (hub-and-spoke):
VPC-A <-> VPC-B                 VPC-A --+
VPC-A <-> VPC-C                 VPC-B --+--> TGW --> On-premises (VPN/DX)
VPC-B <-> VPC-C                 VPC-C --+
(N*(N-1)/2 connections)         (N connections — scales cleanly)
```

### Key Facts

- Regional resource — one TGW per region, all VPCs in that region attach to it
- One attachment per AZ is needed (not per VPC or per subnet)
- Does not allow VPCs with **overlapping CIDRs**
- Can be shared across accounts using **AWS RAM**
- Supports **IPv4 and IPv6**
- Can attach an **IPSec VPN** for on-premises connectivity

### TGW Inter-Region Peering

TGWs in different regions can peer with each other, enabling cross-region VPC communication:

```
TGW (us-east-1) <------- TGW Peering -------> TGW (eu-west-1)
```

Each region needs its own TGW. TGW peering does NOT enable transitive routing through a TGW.

### TGW Connect Attachment

For connecting SD-WAN or third-party virtual appliances running in a VPC:

- Uses **GRE (Generic Routing Encapsulation)** tunnels for high performance
- Uses **BGP** for dynamic routing

### Service Quotas

| Resource | Limit |
|---|---|
| Transit Gateways per account | 5 |
| Transit Gateways per VPC | 5 |
| Attachments per Transit Gateway | 5000 |
| Transit Gateways per DX Gateway | 3 |

---

## VPC Endpoints

VPC Endpoints allow private subnet resources to reach AWS services **without leaving the AWS network** — no NAT Gateway, no IGW required.

### Why Use Them?

- Keeps traffic private and off the internet
- Reduces NAT Gateway data processing costs (traffic to S3/DynamoDB via Gateway endpoint is free)
- Required for compliance scenarios that mandate no internet traversal

### Gateway Endpoints

A gateway configured as a **target in the route table** — no ENI, no DNS change needed.

- Supported services: **S3 and DynamoDB only**
- One gateway endpoint per VPC per service
- Cannot associate a security group
- **Free to use** — no hourly or data transfer charges
- Region-specific — does not support cross-region access

```
Private EC2 → Route Table (pl-xxxxx → vpce-xxx) → S3 (private, stays in AWS network)
```

### Interface Endpoints (AWS PrivateLink)

An ENI with a **private IP address** deployed in your subnet, DNS-based routing to redirect service traffic.

- Supports most AWS services and third-party services
- Create one ENI per AZ for HA
- Can associate a security group
- Pricing: **$0.01/AZ/hr + $0.01/GB** data processed
- Can be accessed from **outside the VPC** (on-premises via DX or VPN) by resolving to the ENI IP directly

### S3 Endpoints — Which to Use?

| Scenario | Recommended Endpoint |
|---|---|
| VPC resources accessing S3 | Gateway Endpoint (free) |
| On-premises resources accessing S3 over DX/VPN | Interface Endpoint |

> **From our discussion:** For on-premises access to S3 via a **Gateway endpoint**, a DNS-based proxy solution is needed (proxy servers in public subnets behind an ELB) because Gateway endpoints only work within the VPC CIDR. Interface endpoints are the preferred solution — simpler, no proxy required, resolves directly to private ENI IPs.

### Endpoint Policies

By default, an endpoint allows full access to the designated service. Attach an **endpoint policy** to restrict access — for example, allow reads only to specific S3 buckets.

### Centralized Endpoint Pattern

Deploy interface endpoints in a **hub VPC**. Spoke VPCs connect to the hub VPC and use its endpoints centrally — reduces the number of endpoints and cost at scale.

---

## IPv6 Egress-Only Internet Gateway

IPv6 addresses are **all public** — unlike IPv4, there is no NAT for IPv6. The Egress-Only Internet Gateway allows IPv6 instances to initiate outbound internet connections while blocking unsolicited inbound connections.

- **Stateful** (tracks connections, like a standard IGW)
- Cannot associate a security group
- Only for **IPv6** traffic — not used for IPv4

```
Private IPv6 EC2 --> Egress-Only IGW --> Internet (outbound only)
Internet --> X (no inbound through Egress-Only IGW)
```

---

## VPC Flow Logs

VPC Flow Logs capture **metadata** about IP traffic flowing through network interfaces in a VPC.

### What is Captured

- Source IP and port
- Destination IP and port
- Traffic status: **ACCEPT** or **REJECT**
- Protocol, packet count, byte count, timestamps

### Scope Options

| Level | What it captures |
|---|---|
| **VPC** | All traffic across the entire VPC |
| **Subnet** | All traffic for all ENIs in a subnet |
| **ENI** | Traffic for one specific network interface |

### Destinations

- **Amazon CloudWatch Logs** — real-time monitoring and alerting
- **Amazon S3** — long-term storage and analysis with Athena/Glue

> **Use cases:** Troubleshooting security group and NACL rules, forensic investigation, compliance auditing, network performance analysis.

---

## AWS Local Zones

An AWS Local Zone is an **extension of an AWS region** placed in close geographic proximity to end users — deployed in a major city not served by a full AWS region.

### Key Facts

- Has its own **internet connection** and supports **Direct Connect**
- Managed by AWS in AWS-operated facilities
- Extend your VPC into a Local Zone by creating a subnet there
- Once extended, existing VPC constructs work: route tables, security groups, NACLs

### Supported Services

EC2, EBS, RDS, ECS, VPC, and more.

### Use Cases

- Real-time gaming
- Live streaming and media production
- AR/VR applications
- Virtual workstations for content creators

### Local Zone vs Wavelength Zone

| Feature | Local Zone | Wavelength Zone |
|---|---|---|
| **Location** | AWS facility near a city | Inside telecom carrier's 5G network |
| **Connectivity** | Internet + Direct Connect | 4G LTE / 5G carrier network |
| **Target users** | End users near a metro area | Mobile 5G device users |
| **Latency target** | Single-digit milliseconds | Ultra-low latency for mobile |

---

## Exam Traps Quick Reference

| Trap | Correct Answer |
|---|---|
| NAT Gateway needs Source/Destination Check disabled? | No — only NAT **Instance** needs this |
| Can a NAT Gateway be used as a Bastion? | No — only NAT Instance can SSH |
| Does VPC Peering support transitive routing? | No — direct peering required for each pair |
| Does VPC Peering support edge-to-edge routing? | No — on-premises cannot route through a peer |
| Gateway Endpoint for S3 works from on-premises? | No — use Interface Endpoint for on-premises |
| Gateway Endpoint cost? | Free |
| IPv6 Egress-Only IGW — allows inbound from internet? | No — outbound only |
| Flow Logs capture packet payload/content? | No — metadata only (IPs, ports, accept/reject) |
| Local Zone — is it inside a carrier's 5G network? | No — that is Wavelength Zone |
| Private NAT Gateway traffic — can it go through IGW? | No — dropped by IGW |

---

*AWS Solutions Architect Associate — VPC Deep Dive Study Notes*

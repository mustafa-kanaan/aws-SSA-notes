# AWS Hybrid Cloud Networking — VPN, Direct Connect, DX Gateway & AWS Wavelength

## Overview

This document covers AWS hybrid cloud networking: Site-to-Site VPN, AWS Direct Connect, VIF types, Direct Connect Gateway, Transit Gateway integration for enterprise scale, and AWS Wavelength for 5G edge computing. These are the core services for building private, secure, and low-latency connectivity between on-premises environments and AWS.

---

## Hybrid Connectivity Overview

| Method | Path | Latency | Security | Cost |
|---|---|---|---|---|
| **Site-to-Site VPN** | Over the internet | Variable | Encrypted (IPSec) | Low |
| **Direct Connect** | Private fiber | Consistent, low | Private but NOT encrypted | Higher |
| **DX + VPN** | DX for transport, VPN for encryption | Consistent | Encrypted | Medium-High |

---

## IPSec and BGP — Foundations

### IPSec

IPSec is the encryption protocol used to secure VPN tunnels. It authenticates and encrypts data packets between two endpoints — host-to-host, host-to-gateway, or gateway-to-gateway (site-to-site).

- Used in AWS Site-to-Site VPN
- Can be established over the internet or any IP-enabled network
- Provides confidentiality, integrity, and authentication

### BGP — Border Gateway Protocol

BGP is a dynamic routing protocol used to exchange routing information between networks (Autonomous Systems). In AWS:

- Used between the AWS VGW/TGW and your on-premises router
- Allows automatic route propagation — no manual static routes needed
- Each AS (Autonomous System) has a unique **ASN (Autonomous System Number)**

> **From our discussion:** Static routing works for simple setups, but BGP is preferred in production because it handles failover automatically by withdrawing routes when a path goes down.

---

## AWS Site-to-Site VPN

### What is it?

An **IPSec-based encrypted tunnel** between your on-premises network and an AWS VPC, routed over the public internet.

### Components

| Component | Description |
|---|---|
| **Virtual Private Gateway (VGW)** | AWS-side VPN endpoint attached to a VPC |
| **Customer Gateway (CGW)** | Represents your on-premises router in AWS |
| **VPN Connection** | The IPSec tunnel between VGW and CGW |
| **Tunnel** | Two tunnels created per VPN connection for HA |

```
On-premises Router (CGW) <=== IPSec Tunnel ===> VGW --> VPC
                          (over internet)
```

### Key Characteristics

- **Internet-based** — latency varies with internet conditions
- **NAT-Traversal** supported — works even if on-premises router is behind NAT
- Two tunnels per connection — AWS recommends keeping both active for HA
- Pricing: **$0.05/hr** per VPN connection + data transfer out charges

### Accelerated VPN

Uses **AWS Global Accelerator** to route VPN traffic over the AWS backbone instead of the internet — reduces latency and improves stability. Configured at VPN creation time.

---

## AWS Direct Connect (DX)

### What is it?

A **dedicated private fiber connection** between your on-premises data center and AWS, obtained through a telecom provider at an AWS Direct Connect location.

### Key Characteristics

- Private link — not over the internet
- Consistent, low latency and higher bandwidth
- **Not encrypted by default** — private but not secure without an additional IPSec layer
- Longer lead time to provision — typically a few months
- More costly than VPN

### Connection Types

| Type | Speed | Notes |
|---|---|---|
| **Dedicated Connection** | 1, 10, or 100 Gbps | Direct physical port at DX location |
| **Hosted Connection** | 50 Mbps – 10 Gbps | Ordered through an AWS Partner |

### Link Aggregation Groups (LAGs)

You can aggregate up to **4 Direct Connect connections of the same speed** into a logical LAG — they behave as a single logical link with combined bandwidth.

### DX + VPN for Encryption

Since DX is not encrypted by default, you can run an **IPSec VPN over the DX connection** to achieve both private, consistent connectivity and encryption.

```
On-premises --> DX (private path) --> IPSec VPN Tunnel --> VGW --> VPC
```

---

## Virtual Interfaces (VIFs)

A **Virtual Interface (VIF)** is the logical connection created on top of a physical Direct Connect connection. One physical DX cable can carry multiple VIFs, each identified by a unique **VLAN ID**.

```
Physical DX Cable
        |
        |-- VLAN 100 --> Private VIF  --------> VGW --> 1 VPC
        |
        |-- VLAN 200 --> Transit VIF  --> DX Gateway --> TGW --> many VPCs
        |
        +-- VLAN 300 --> Public VIF   --------> S3, DynamoDB, CloudWatch...
```

### The 3 VIF Types

#### Private VIF

Connects on-premises directly to **one VPC** via a VGW.

- Use for: simple setup, single VPC
- Traffic reaches: private resources in one VPC (EC2, RDS, etc.)
- Limitation: one VPC only — does not scale

```
On-premises --> DX --> Private VIF --> VGW --> Single VPC
```

#### Transit VIF

Connects on-premises to a **DX Gateway**, which routes to **Transit Gateways** across multiple regions and accounts.

- Use for: enterprise scale — multiple VPCs, regions, accounts
- Requirement: must use Transit VIF (not Private VIF) when attaching to a TGW via DX Gateway

```
On-premises --> DX --> Transit VIF --> DX Gateway --> TGW --> many VPCs
```

#### Public VIF

Connects on-premises to **AWS public service endpoints** (S3, DynamoDB, CloudWatch, etc.) over the AWS backbone — traffic never touches the internet.

```
On-premises --> DX --> Public VIF --> S3, SQS, CloudWatch...
```

### VIF Selection Guide

| Scenario | VIF Type |
|---|---|
| Accessing one VPC only | Private VIF |
| Accessing multiple VPCs or regions at scale | Transit VIF |
| Accessing AWS public services (S3, DynamoDB) | Public VIF |

### VIF Configuration Steps

```
AWS Console --> Direct Connect --> Virtual Interfaces --> Create VIF

1. Choose VIF type: Private / Transit / Public
2. Enter: VIF name, Connection, VLAN ID, BGP ASN, BGP auth key (optional)
3. Attach to:
   - Private VIF  --> VGW
   - Transit VIF  --> DX Gateway
   - Public VIF   --> no attachment needed
4. Configure matching VLAN + BGP on your on-premises router
```

### Service Quotas

| Resource | Limit |
|---|---|
| VIFs per Direct Connect connection | 50 |
| Transit VIFs per Direct Connect connection | 1 |
| VGWs per Direct Connect Gateway | 10 |
| Transit Gateways per Direct Connect Gateway | 3 |

---

## Direct Connect Gateway (DX Gateway)

### What is it?

A **globally scoped** AWS resource that allows a **single DX connection** to reach VPCs across **multiple AWS regions and accounts**.

```
Without DX Gateway:
On-premises --> DX1 --> VPC (us-east-1)
            --> DX2 --> VPC (eu-west-1)      <-- separate expensive connections
            --> DX3 --> VPC (ap-southeast-1)

With DX Gateway:
                             +--> VPC (us-east-1)
On-premises --> DX --> DXGW -+--> VPC (eu-west-1)
                             +--> VPC (ap-southeast-1)
```

### Key Facts

| Feature | Detail |
|---|---|
| **Scope** | Global — not region-specific |
| **Max VGWs/TGWs** | 10 per DX Gateway |
| **Cross-account** | Yes |
| **Cross-region** | Yes |
| **VPC-to-VPC routing** | No — on-premises to VPC only |
| **Required VIF** | Transit VIF (when used with TGW) |

> **Important:** DX Gateway does NOT enable VPC-to-VPC communication. It only connects on-premises to VPCs. For VPC-to-VPC, use VPC Peering or Transit Gateway.

---

## Enterprise-Scale Hybrid Architecture

### Scenario

Two on-premises data centers need to reach 50 VPCs across 3 AWS regions. All VPCs within each region must communicate. Cross-region VPC communication is also required.

### Solution

```
On-premises DC-1 --> DX --> Transit VIF --> DX Gateway --+
                                                         |
On-premises DC-2 --> DX --> Transit VIF --> DX Gateway --+
                                                         |
                            +----------------------------+
                            |
                            +--> TGW (us-east-1)       <--> VPC-1 ... VPC-20
                            |          |
                            |          | TGW Inter-Region Peering
                            |          v
                            +--> TGW (eu-west-1)       <--> VPC-21 ... VPC-35
                            |          |
                            |          | TGW Inter-Region Peering
                            |          v
                            +--> TGW (ap-southeast-1)  <--> VPC-36 ... VPC-50
```

### Services Used

| Need | Solution |
|---|---|
| Private DC-to-AWS connectivity | Direct Connect |
| Single DX reaching multiple regions | DX Gateway |
| All VPCs in one region communicating | Transit Gateway |
| Cross-region VPC communication | TGW Inter-Region Peering |
| DX to DX Gateway to TGW | Transit VIF |

> **From our discussion:** VPCs within each region communicate through their regional TGW. Cross-region communication uses TGW inter-region peering. The two DX Gateway instances connect both data centers to all three regional TGWs — giving each DC access to all 50 VPCs.

---

## High Availability for Direct Connect

AWS recommends two approaches:

1. **Two DX connections from two different DX locations** — for physical path redundancy
2. **DX as primary + VPN as backup** — VPN activates automatically via BGP if DX fails

```
On-premises --> DX (primary)  --> VGW --> VPC
            --> VPN (backup)  --> VGW --> VPC  (activates if DX fails)
```

---

## AWS Wavelength

### The Problem

5G networks deliver 10x faster performance than 4G — but only if applications are physically close to the user. If the app runs in a standard AWS region, traffic must travel from the 5G device → telecom tower → internet → AWS region → back, which eliminates the 5G latency benefit.

```
Without Wavelength:
5G Device --> Telecom Tower --> Internet --> AWS Region --> back
                                  ^--- latency added here

With Wavelength:
5G Device --> Telecom Tower --> Wavelength Zone (inside carrier network)
                                    ^--- app runs HERE, ultra-low latency
```

### What is AWS Wavelength?

AWS Wavelength deploys **standard AWS compute and storage services** (EC2, EBS, ECS, EKS) **inside a telecom carrier's 5G network** — at the edge, physically co-located with the carrier's infrastructure.

A **Wavelength Zone (WZ)** is a zone inside a carrier location (e.g., Verizon, Vodafone, KDDI) where you deploy your workloads.

### Key Facts

| Feature | Detail |
|---|---|
| **Opt-in required** | Yes — must enable Wavelength Zones per account |
| **Connected to** | A parent AWS region (for access to other AWS services) |
| **VPC extension** | Extend your existing VPC into one or more WZs |
| **Entry/exit point** | Carrier Gateway |
| **Internet inbound** | No — not through Carrier Gateway |
| **Internet outbound** | Yes — outbound only |

### Carrier Gateway

The Carrier Gateway is the entry/exit point for a Wavelength Zone — functionally similar to an Internet Gateway but for the carrier network:

- Handles **bidirectional traffic** between 5G devices and the WZ
- Performs **NAT** — maps WZ instance private IPs to Carrier IPs (like Elastic IP in normal AZs)
- Must be set as the **default gateway (0.0.0.0/0)** in the WZ subnet route table
- **No inbound connections from the internet** — only carrier 4G/5G traffic is allowed in

```
5G Device
    |  (bidirectional — carrier network only)
Carrier Gateway
    |
WZ Subnet --> EC2 in Wavelength Zone
    |
Parent AWS Region (via private connection for other AWS services)
```

### Supported Services in Wavelength Zones

EC2 and Auto Scaling, EBS, ECS, EKS, SSM, CloudWatch, CloudTrail, CloudFormation

### Use Cases

- Connected vehicles — real-time HD maps, intelligent driving
- Real-time cloud gaming
- Live high-speed media streaming, AR/VR
- Smart factories, IoT + ML at the edge
- Healthcare applications requiring real-time responses

### Local Zone vs Wavelength Zone

| Feature | Local Zone | Wavelength Zone |
|---|---|---|
| **Location** | AWS-managed facility near a city | Inside telecom carrier's 5G network |
| **Connectivity** | Internet + Direct Connect | 4G LTE / 5G carrier network |
| **Target users** | End users near a metro area | Mobile 5G device users |
| **Latency target** | Single-digit milliseconds | Ultra-low latency for mobile |
| **Inbound internet** | Yes | No — carrier traffic only |

---

## Exam Traps Quick Reference

| Trap | Correct Answer |
|---|---|
| Can you use a Private VIF to connect to a Transit Gateway? | No — must use Transit VIF |
| Does DX Gateway allow VPC-to-VPC communication? | No — on-premises to VPC only |
| Can one DX connection reach multiple regions? | Yes — via DX Gateway |
| Is Direct Connect encrypted by default? | No — private but not encrypted |
| How do you encrypt DX traffic? | Run IPSec VPN over the DX connection |
| Max VGWs/TGWs per DX Gateway? | 10 |
| Max Transit VIFs per DX connection? | 1 |
| Does Carrier Gateway allow inbound internet traffic? | No — outbound only |
| Must you opt in to use Wavelength Zones? | Yes |
| Does one Transit Gateway serve one VPC or one region? | One region — all VPCs in that region attach to it |
| Can TGW peer across regions? | Yes — TGW inter-region peering |
| DX lead time? | Typically a few months (long lead time) |

---

*AWS Solutions Architect Associate — Hybrid Cloud Networking Study Notes*

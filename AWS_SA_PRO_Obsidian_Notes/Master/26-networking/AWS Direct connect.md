
---
tags: [networking, hybrid, security, service:direct-connect, domain:hybrid-networking, concept:private-vif, concept:public-vif, concept:macsec, concept:site-link]
---

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]] Study Guide Highlights**

- **Dynamic Routing:** [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] allows for automatic failover. If a primary path fails, [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] detects the loss and reroutes traffic to a secondary path (e.g., a backup Direct Connect link or a [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]).

- **ASN:** You must configure Autonomous System Numbers (ASNs) for the peering session. AWS assigns specific CIDRs for IPv6 [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] peering, while IPv4 addresses can be customer-configured or AWS-generated.

**5. [[appsync|Security]] Features**

Direct Connect is a private line, but traffic is not encrypted by default. You can secure the “process” using the following methods:

- **MACsec (Layer 2):** For dedicated connections (10 Gbps, 100 Gbps, 400 Gbps), you can enable MACsec encryption. This provides point-to-point encryption between your router and the AWS edge device.

- **[[IPSec VPN]] (Layer 3):** For end-to-end encryption, you can establish an [[IPSec VPN]] tunnel _over_ a Direct Connect Public or Transit VIF. This combines the performance of a dedicated line with the encryption of a [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]].

**6. Advanced Features: SiteLink**

The process can also be configured to bypass AWS regions entirely for site-to-site data transfer. By enabling **SiteLink** on your VIFs, you can route traffic between your different on-premises data centres using the AWS global backbone, effectively using AWS as your WAN provider.

**— Private** and **Public Virtual Interfaces (VIFs)** used in [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]].

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]] uses industry-standard 802.1Q VLANs (Virtual Local Area Networks) to share a single physical connection for multiple purposes. These VLANs are configured as “Virtual Interfaces” (VIFs) to separate traffic routing logic.

**1. Private Virtual Interface (Private VIF)**

- **Purpose:** A Private VIF allows you to connect your on-premises infrastructure to **private IP addresses** within your Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

- **Function:** It effectively makes your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] function as a logical Layer-3 extension of your on-premises data centre. This allows you to connect directly to resources like [[ec2]] instances or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] databases running in private subnets without needing public IP addresses or internet gateways.

  - **Connectivity Options:**
    - **Direct to [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]:** You can connect a Private VIF directly to a **Virtual Private Gateway (VGW)** attached to a single [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. This provides a 1:1 connection.
    - **Direct Connect Gateway:** You can attach a Private VIF to a **Direct Connect Gateway (DXGW)**. This allows a single Private VIF to connect to up to 10 VPCs across different AWS Regions and accounts.

- **Use Case:** This is the standard choice for hybrid architectures where you need high-throughput, low-latency access to internal corporate applications hosted in AWS.

**2. Public Virtual Interface (Public VIF)**

- **Purpose:** A Public VIF allows your on-premises network to reach **AWS public endpoints** over your dedicated Direct Connect line, rather than traversing the public internet.

- **Function:** It provides access to services that are publicly addressable, such as **Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]**, **Amazon [[dynamodb]]**, and **public IP addresses of [[ec2]] instances**.

  - **Routing & [[appsync|Security]]:**
    - **[[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]]:** Because these services use public IP addresses, you must use a unique public IPv4 CIDR that you own (or have assigned) for the [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] peering session.
    - **Traffic Scope:** While this interface connects to public IPs, the traffic remains on the AWS global backbone. However, because it connects to the “public” AWS zone, any AWS public resource (including those owned by other customers) could theoretically reach your gateway. Therefore, it is critical to implement a **firewall** on your on-premises router to filter traffic.

  - **Use Case:**
    - Accessing [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets or [[dynamodb|DynamoDB tables]] with higher performance and consistency than the public internet allows.

**Behaviour:** AWS treats the bundle as a single connection. If one physical cable is cut, traffic automatically shifts to the remaining active cables in the group without [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] recalculation, providing sub-second failover at the physical layer.
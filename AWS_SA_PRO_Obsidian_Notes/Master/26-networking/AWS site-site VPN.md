
---
tags: [networking, hybrid, security, service:site-to-site-vpn, domain:hybrid-networking, concept:customer-gateway, concept:ecmp, concept:redundancy, concept:failover]
---

**1. Resources**

- **Customer Gateway (CGW):** A resource in AWS representing your physical on-premises device. You define its public IP and [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] Autonomous System Number (ASN).

- **[[Transit gateway]] (TGW):** A regional hub that allows you to consolidate multiple [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections. Unlike VGW, it supports **Equal Cost Multipath (ECMP)** to aggregate bandwidth across multiple tunnels.

**2. Architecture Patterns**

- **One-to-One (VGW):** Best for simple setups connecting a single remote site to a single [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. It is easy to set up but hard to manage at scale.

- **Hub-and-Spoke ([[Transit gateway]]):** The recommended pattern for scaling. A single [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection to a [[Transit gateway]] gives access to hundreds of attached VPCs. It simplifies management and enables higher throughput via ECMP.

- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] CloudHub:** Uses a VGW to connect multiple remote branch offices in a hub-and-spoke model. Sites can communicate with each other through the AWS VGW using [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] re-advertisement.

**3. Performance & Acceleration**

- **Throughput Limits:** Standard tunnels are limited to **1.25 Gbps**.

- **Scaling Bandwidth:** To exceed 1.25 Gbps, you must use **[[Transit gateway]]** with dynamic routing ([[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]]) to enable **ECMP**. This distributes traffic across multiple active tunnels.

- **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Accelerated Site-to-Site VPN]]:** Uses [[AWS Global Accelerator 1]] to route traffic from your on-premises device to the closest AWS edge location. This avoids internet congestion, reducing latency and packet loss.

**4. High Availability (HA)**

- **Double Tunnels:** Every Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection includes **two tunnels**, each terminating in a different AWS Availability Zone. AWS performs routine maintenance on these tunnels, so configuring both on your customer gateway is critical for redundancy.

- **Failover Testing:** It is a best practice to regularly simulate failures (e.g., by disabling one tunnel) to ensure your router automatically switches traffic to the second tunnel.

**5. Routing**

- **Dynamic ([[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]]):** Preferred for production. It allows for automatic failover and route propagation. [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] is required for ECMP.

- **Static:** Requires manual route updates. It does not support ECMP or automatic failover for changes in network topology.

**Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]: TGW vs. VGW**

This comparison is critical for scaling.

- **Virtual Private Gateway (VGW) - “The Legacy/Simple Model”**

  - **Architecture:** A 1:1 connection. You attach a VGW to a _single_ [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

  - **[[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|Limitations]]:** If you have 100 VPCs, you need to create and manage 100 separate [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections (one per VGW). This effectively creates a management nightmare and limits transitive routing ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] A cannot talk to [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] B via the VGW).

  - **Bandwidth:** Limited to 1.25 Gbps per tunnel.

- **[[transit-gateway|AWS Transit Gateway]] (TGW) - “The Hub Model”**

  - **Architecture:** Acts as a regional router. You create _one_ [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection to the [[Transit gateway]]. You then “attach” thousands of VPCs to that same [[Transit gateway]].

  - **Routing:** Traffic from any [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] is routed to the TGW, which then sends it down the single [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] pipe to on-premises.

  - **Performance (ECMP):** TGW supports **Equal Cost Multipath**. You can configure multiple [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] tunnels to the same TGW and advertise the same prefixes. TGW will aggregate the bandwidth. For example, 4 tunnels x 1.25 Gbps = 5 Gbps total throughput. VGW does not support this aggregation for a single flow.
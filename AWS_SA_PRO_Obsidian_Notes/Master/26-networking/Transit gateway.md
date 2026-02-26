
---
tags: [networking, hybrid, security, service:transit-gateway, domain:hybrid-networking, concept:hub-and-spoke, concept:ecmp, concept:attachment, concept:route-table]
---

1. A _transit gateway_ is a network transit hub that you can use to interconnect your virtual private clouds (VPCs) and on-premises networks. You can attach the following to a transit gateway:

- One or more VPCs
- A Connect SD-WAN/third-party network appliance
- An [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]] gateway
- A peering connection with another transit gateway
- A [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection to a transit gateway

1. [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] attachment with [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] for an [[transit-gateway|AWS Transit Gateway]]. This allows [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] equal-cost multipathing (ECMP) to be used which can load balance traffic across multiple [[ec2]] instances. This is the only solution that provides the ability to horizontally scale the outbound internet traffic across multiple appliances with HA across AZ

**[[transit-gateway|AWS Transit Gateway]] Study Guide Highlights**

**1. Core Architecture: Hub-and-Spoke**

• **Function:** [[transit-gateway|AWS Transit Gateway]] acts as a highly available, scalable regional router. It simplifies network topology by allowing you to connect thousands of VPCs and on-premises networks to a single gateway, rather than managing a complex mesh of peerings.

• **Scope:** It is a **Regional** resource. To connect networks across different AWS Regions, you use **[[transit-gateway|Transit Gateway Peering]]**, which sends traffic encrypted over the AWS global backbone.

**2. Connectivity Types (Attachments)**

• **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Attachment:** Connects VPCs to the gateway.

• **[[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] Attachment:** Standard IPsec Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections. Supports **ECMP** (Equal Cost Multipath) to aggregate throughput across multiple tunnels.

• **Direct Connect Gateway:** Connects [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]] via a **Transit VIF**, enabling high-speed private connectivity from on-premises.

• **Transit Gateway Connect:** Enables native integration with SD-WAN appliances using **GRE tunnels** and **[[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]]** over existing attachments ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] or Direct Connect). It supports higher bandwidth per attachment compared to standard [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|VPNs]].

**3. Routing Concepts**

• **Route Tables:** Transit Gateway uses its own route tables to control traffic flow. You can separate traffic (e.g., Production vs. Development) by using different route tables for different attachments.

• **Associations:** A [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] attachment is _associated_ with a specific route table, which determines where traffic _from_ that [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] goes.

• **Propagations:** Routes from an attachment are _propagated_ into a route table, allowing other associated attachments to learn how to reach it.

• **Appliance Mode:** A critical setting for stateful traffic inspection. It ensures that bidirectional traffic flows through the same Availability Zone to maintain flow symmetry for firewalls.

**4. Centralized [[appsync|Security]] Patterns**

• **Centralized Egress:** You can route all internet-bound traffic from spoke VPCs to a central "Egress [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]" containing NAT Gateways or Firewalls, reducing cost and complexity.

• **Traffic Inspection:** Transit Gateway is the core component for "East-West" (VPC-to-VPC) inspection. It routes traffic to a central "[[appsync|Security]] [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]" housing **[[network-and-dns-firewall|AWS Network Firewall]]** or **Gateway Load Balancers** before sending it to the final destination.

**5. Scalability & Limits**

• **Attachments:** Up to **5,000** attachments per Region.

• **Bandwidth:** [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] attachments can burst up to **100 Gbps**. Connect attachments (SD-WAN) support up to **20 Gbps** per attachment.
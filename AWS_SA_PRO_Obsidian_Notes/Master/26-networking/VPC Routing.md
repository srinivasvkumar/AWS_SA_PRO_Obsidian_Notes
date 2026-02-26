

Based on the provided sources, here is a detailed explanation of **VPC Routing**, covering the core mechanisms for connecting VPCs to each other, to the internet, and to on-premises networks.

**1. VPC-to-VPC Routing**

There are two primary models for routing traffic between Virtual Private Clouds (VPCs):

• **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering (Mesh Model)**

    ◦ **Architecture:** This is a one-to-one networking connection between two VPCs. It enables them to communicate using private IP addresses as if they were on the same network.

    ◦ **Routing Logic:** You must update the route tables in _both_ VPCs. For example, in [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] A's route table, you add a route for [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] B's CIDR block pointing to the peering connection ID (`pcx-xxxxx`).

    ◦ **[[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|Limitations]]:** It does not support transitive routing. If [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] A is peered with [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] B, and [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] B is peered with [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] C, [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] A cannot talk to [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] C through [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] B. This means a "full mesh" of peerings is required for full connectivity, which becomes difficult to manage at scale (limit of 125 active peers per [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]).

• **[[transit-gateway|AWS Transit Gateway]] (Hub-and-Spoke Model)**

    ◦ **Architecture:** [[Transit gateway]] acts as a regional cloud router. You attach your VPCs to this central hub using "[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Attachments."

    ◦ **Routing Logic:** [[Transit gateway]] has its own route tables.

        ▪ **Associations:** Each [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] attachment is associated with a [[Transit gateway]] route table. This determines where traffic _from_ that [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] is routed.

        ▪ **Propagations:** Routes from a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] can be automatically propagated into the [[Transit gateway]] route table, allowing other attachments to learn how to reach that [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

        ▪ **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|VPC Route Tables]]:** In your individual [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|VPC route tables]], you simply add a route for all other internal network traffic (e.g., `10.0.0.0/8`) pointing to the [[Transit gateway]] ID (`tgw-xxxxx`). This dramatically simplifies routing compared to managing hundreds of peering connections.

**2. Routing to the Internet (Egress)**

Routing traffic from your VPCs to the internet typically follows one of two patterns:

• **Decentralized Egress:**

    ◦ Each [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] has its own **Internet Gateway (IGW)** for public subnets or **[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]]** for private subnets.

    ◦ **Route Table:** Public subnets have a route `0.0.0.0/0` pointing to the IGW. Private subnets have a route `0.0.0.0/0` pointing to the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] within their Availability Zone.

• **Centralized Egress (Hub-and-Spoke):**

    ◦ To reduce costs and improve [[appsync|security]] monitoring, [[organizations]] often use a dedicated **"Egress [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]"**.

    ◦ **Routing Path:**

        1. **Spoke [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]:** The default route (`0.0.0.0/0`) in the spoke [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] points to the [[Transit gateway]].

        2. **[[Transit gateway]]:** The TGW route table sends this traffic to the "Egress [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]" attachment.

        3. **Egress [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]:** Traffic arrives in the Egress [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], where it is routed to a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] or Network Firewall, and finally to an Internet Gateway.

**3. Hybrid Routing (On-Premises to [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]])**

Connecting your on-premises network to AWS involves specific routing protocols:

• **Dynamic Routing ([[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]]):**

    ◦ Both **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|AWS Site-to-Site VPN]]** and **[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]]** use the **Border Gateway Protocol ([[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]])** to exchange routing information automatically.

    ◦ **How it works:** Your on-premises router advertises your internal IP prefixes to AWS, and AWS advertises your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] prefixes back to you. If a link fails (e.g., a primary Direct Connect), [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] detects the failure and automatically reroutes traffic to a backup path (e.g., a backup [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]).

• **Direct Connect Gateway (DXGW):**

    ◦ This is a global resource that allows a single Direct Connect connection to route traffic to VPCs in multiple AWS Regions. The DXGW holds the route information and propagates it to the attached Virtual Private Gateways (VGWs) or Transit Gateways.

**4. Advanced Routing Concepts**

• **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|Ingress Routing]]:** You can use **[[gwlb|Gateway Load Balancers (GWLB)]]** to inspect traffic coming _into_ your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. By updating [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|VPC route tables]] (using "[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|Ingress Routing]]" features), you can force traffic arriving at an Internet Gateway to pass through a firewall appliance before reaching your application servers.

• **Route Priorities:** When multiple paths exist (e.g., a Direct Connect and a [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]), the "Hybrid Networking Lens" [[vpc-flow-logs|notes]] that you can use [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] attributes like `AS_Path` prepending to influence which path AWS prefers for outbound traffic.

• **Null Routes (Blackholes):** For [[appsync|security]] segmentation, you can add "blackhole" or null routes in [[Transit gateway]] tables to explicitly drop traffic between specific VPCs (e.g., preventing Dev from talking to Prod)

1. [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|Advanced VPC Routing]]: CIDR Overlap**

When two networks use the same IP address range (e.g., both use `10.0.0.0/16`), standard routing fails because the router cannot distinguish between the local network and the destination. The documents provide two specific architectural patterns to solve this:

• **Private [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] (The "Hide" Method):**

    ◦ **Scenario:** You have two VPCs ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] A and [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] B) that both use `100.64.0.0/16` but need to communicate.

    ◦ **Configuration:**

        1. Add a **secondary, non-overlapping CIDR** (routable subnet) to both VPCs (e.g., `10.0.1.0/24` to [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] A and `10.0.2.0/24` to [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] B).

        2. Deploy a **Private [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]]** in [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] A's routable subnet.

        3. Deploy an **[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]]** in [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] B's routable subnet.

    ◦ **Traffic Flow:** When an instance in [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] A wants to talk to [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] B, it sends traffic to the Private [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]]. The [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] replaces the source IP (the overlapping one) with its own unique IP (`10.0.1.x`). [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] B sees the traffic coming from the unique NAT IP, not the overlapping range, allowing the return traffic to route correctly.

• **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]] (The "Projection" Method):**

    ◦ **Scenario:** A service provider wants to offer a service to a consumer, but their [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] CIDRs might overlap.

    ◦ **Configuration:** The provider places a [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Network Load Balancer (NLB)]] in front of their service. The consumer creates an **Interface [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Endpoint**.

    ◦ **Traffic Flow:** AWS projects the service into the consumer's [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] as an Elastic Network Interface (ENI) with a valid IP from the _consumer's_ subnet. The consumer connects to this local ENI. AWS handles the routing to the provider, completely ignoring the IP overlap issues.

### Architecture Diagram
![[AWS_VPC_Routing_CIDR_1.png]]


### Architecture Diagram
![[AWS_VPC_Routing_CIDR_2.png]]


### Architecture Diagram
![[AWS_VPC_Routing_CIDR.png]]


### Architecture Diagram
![[AWS_VPC_Routing_Ingress.png]]


### Architecture Diagram
![[AWS_VPC_Routing.png]]


### Architecture Diagram
![[AWS_VPC_Routing_1.png]]

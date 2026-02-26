---
aliases:
- "AWS Site-to-Site VPN"
- "Accelerated Site-to-Site VPN"
- "Client VPN"
- "VPNs"
---

# VPNs

# [[IPSec VPN]] Fundamentals

- IPSEC is a group of protocols
- Their aim is to set up secure tunnels across insecure networks. Example: connect two secure networks (peers) over the public internet
- IPSEC provides [[api-gateway|authentication]]
- Any traffic transferred through IPSEC is encrypted
- IPSEC is using [[kms|asymmetric encryption]] to exchange symmetric keys and use these of ongoing encryption
- IPSEC has 2 main phases:
    - IKE (Internet Key Exchange) Phase 1
        - It is slow and heavy
        - It is a protocol of how keys are exchanged
        - It has 2 versions v1 (older) and v2 (newer)
        - Uses [[kms|asymmetric encryption]] to agree on and create a shared symmetric key
        - The end of this phase is an IKE SA ([[appsync|security]] association) phase 1 tunnel
        ![IPSEC phase 1 architecture](IPSECvpn2.png)
    - IKE Phase 2
        - It is faster and more agile
        - Uses the keys agreed in phase 1
        - Is concerned with agreeing on encryption method and keys used for bulk data transfer
        - The end result is an IPSEC SA phase 2 tunnel (runs over phase 1)
        ![IPSEC phase 2 architecture](IPSECvpn3.png)
- There are two types of VPNs - how they match traffic:
    - Policy based VPNs: rule sets match traffic, we can have different rules for different types of traffic
    - Route based VPNs: target matching is done based on prefix. We have a single pair of [[appsync|security]] associations for each network prefix (less functionality, much simpler to set up)
    ![Route vs Policy Based VPNs](IPSECvpn4.png)

## AWS Site-to-Site VPN

- A Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] is a logical connections between a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] and an on-premise network running over the public internet. The connection is encrypted using IPSec
- Can be fully HA if it is implemented correctly
- It is quick to provision, it can be provisioned in less than an hour (contrast to DX)
- Components involved in creating a [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection:
    - **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]**
    - **Virtual Private Gateway (VGW)**: it is a gateway object which can be the target of one or more rules in a Route Tables. It can be associated to a single [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]
    - **Customer Gateway (CGW)**: can refer to 2 different things:
        - Often is referred to the logical configuration in AWS
        - Physical on-premises router which the [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connects to
    - **[[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] Connection** itself: the connection linking the VGW from the AWS to the CGW
- Static vs Dynamic [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]:
    - **Static [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]**:
        - Uses static network configuration: static routes are added to the route tables AWS side, static networks has to be identified on the [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection on-premise side. 
        - It is simple, it just uses IPSec, works anywhere, having limitation on terms of load-balancing and multi-connection failover
    - **Dynamic [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]**:
        - Uses [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/bgp|bgp]] protocol, if customer router does not support [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/bgp|bgp]], we can not use dynamic VPNs
        - [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/bgp|bgp]]: allows routing on the fly, allows multiple links to be used at once between the same locations. Allows using HA available architectures
        - Static routes can still be added to the route tables manually
        - Route propagation: if enabled means that routes are added ro the Route Table automatically
- Considerations for [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]:
    - Speed Limitation for [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] with 2 tunnels: *1.25 Gbps*, AWS limitation. Customer router limitation might also apply
    - Latency considerations: can be inconsistent if the traffic goes through the public internet
    - Cost: hourly cost for outgoing traffic, on-premises data caps might also apply
    - Speed of setup: can done very quickly, within hours or less; IPSec is supported by a wide variety of devices, [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/bgp|bgp]] support is less common. VPNs are always quicker to setup then any other private connection technologies
    - VPNs can be used for Direct Connect backup or they can be used over the Direct Connect for adding a layer of encryption

### Accelerated Site-to-Site VPN

- Performance enhancement for AWS Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] that uses the AWS global network, the same network used for Global Accelerator and [[cloudfront]]
- Using a classic Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]], the traffic goes through the public internet. In order to avoid this, some companies use a Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] over Direct Connect. Direct Connect offers more better performance, but at a higher cost. Since DX is not an option for everybody, accelerated Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] was created to improve performance compared to classic Site-to-Site VPNs
- Accelerated Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] architecture:
![Accelerated Site-to-Site VPN](AWS_SA_PRO_Obsidian_Notes/Master/03-networking/images/AcceleratedS2SVPN1.png)
- Acceleration can be enabled when creating a [[Transit gateway]] attachment only! Not compatible with VPNs using Virtual Private Gateways (VGW)
- Accelerated Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] has a fixed accelerator cost fee and a data transfer fee

## Client VPN

- Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] is generally used for one or more business premisses to connect to AWS VPCs. ClientVPN is similar, but instead of sites connecting to AWS, we have individual clients
- A ClientVPN is a managed implementation of OpenVPN
- Any client device which can use the OpenVPN software is supported
- Architecturally we connect to a Client [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] endpoint which can be associated with one [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] and with one ore more Target Networks (high availability)
- Client [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] [[billing]] is based on network association and hourly fee for usage
- Client [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] setup:
    - We crate a Client [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] Endpoint and associate it with a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] and one or more subnets from the [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]
    - This association places an ENI into the subnets associated
    - We can only pick one subnet per AZ
- Client [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] can use many different methods of [[api-gateway|authentication]] (certificates, [[cognito]] User Pool, Federated Identities,  AWS Directory Service)
- We associate a route table to the Client [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] Endpoint in order to set up routing and connectivity (to internet via NAT Gateways, other VPCs with peering, etc.)
- This route table is pushed to any client which connects to the Client [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] Endpoint
- The default behavior is the Client [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] route table replaces any local routes on the client, meaning the client devices can not access anything locally on their local network without having communication going through the Client [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] Endpoint
- We can use split tunnel [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]], meaning that any routes from the Client [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] Endpoint are added to local client route tables. This solves the problem with the default behavior
- Split tunnel is not the default behavior. It must be enabled by the user, otherwise all the data (including connection to the public internet) will go via the tunnel

### Architecture Diagram
![[AWS_Site_to_Site_VPN.png]]


### Architecture Diagram
![[AWS_Site_to_Site_VPN_HA.png]]


### Architecture Diagram
![[Site2Site_VPN.png]]


### Architecture Diagram
![[AWS_VPN_2_Site-to-Site_VPN_2.png]]


### Architecture Diagram
![[AWS_VPN_1.png]]


### Architecture Diagram
![[AWS_VPN_2_Site-to-Site_VPN_3.png]]


### Architecture Diagram
![[AWS_VPN_2_Site-to-Site_VPN_02.png]]


### Architecture Diagram
![[AWS_VPN_2_Site-to-Site_VPN_1.png]]


### Architecture Diagram
![[AWS_VPN_2_Site-to-Site_VPN_0.png]]


### Architecture Diagram
![[AWS_VPN_2_Site-to-Site_VPN_01.png]]


### Architecture Diagram
![[IPSEC_VPN_Route_vs_Policy.png]]


### Architecture Diagram
![[AWS_DX_Direct_Connect_Public_VIF_1_VPN.png]]


### Architecture Diagram
![[AWS_VPN_2_Split_tunnel.png]]


### Architecture Diagram
![[AWS_VPN_2_Site-to-Site_VPN.png]]


### Architecture Diagram
![[AWS_DX_Direct_Connect_Public_VIF_VPN.png]]


### Architecture Diagram
![[IPSEC_VPN_phase2.png]]


### Architecture Diagram
![[AWS_DX_Direct_Connect_Private_VIF_1_VPN.png]]


### Architecture Diagram
![[IPSEC_VPN_phase1.png]]


### Architecture Diagram
![[AWS_DX_Direct_Connect_BGP_VLAN_1_VPN.png]]


### Architecture Diagram
![[AWS_VPN_2_Site-to-Site_VPN_VGW_03.png]]

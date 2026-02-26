---
aliases:
- "AWS Transit Gateway"
- "Transit Gateway - Deep Dive"
- "Transit Gateway Isolated Routing"
- "Transit Gateway Peering"
---

# AWS Transit Gateway

- It is a network transit hub which connects VPCs to each other and to on-premise networks using Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|VPNs]] and Direct Connects
- It is designed to reduce the network architecture complexity in AWS
- It is a network gateway object, it is HA and scalable
- Attachments: we create attachments in order for the TGW to connect to VPCs and on-premise networks. Valid attachments are:
    - [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] attachments
    - Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|vpn]] attachments
    - Direct Connect Gateway attachments
- Attachments are configured in each subnet of the connected VPCs
- We can also peer transit gateways across cross regions and/or cross accounts
- We can also attach transit gateways to the DX connections
- [[Transit gateway]] Considerations:
    - Supports transitive routing: single [[Transit gateway]] with multiple attachments using route tables
    - Can be used to create global networks with peering
    - We can share transit gateways using AWS [[ram]]
    - Transit Gateways offer less complex architectures compared to [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] peering solutions

## Transit Gateway - Deep Dive

- [[Transit gateway]] is a hub-and-spoke architecture, it can connect to various types of networking objects within AWS
- Integration with Direct Connect:
    - A Transit VIF is required which goes through a DX Gateway
    - The DX Gateway can be attached to the [[Transit gateway]] with a [[Transit gateway]] Attachment
    - 1 DX Gateway can be attached to 3 Transit Gateways
- [[Transit gateway]] has a default route table which is populated from the attachments:
    - For the VPCs we have the CIDR ranges of these VPCs
    - For [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|VPNs]] we have the routes learned via [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/bgp|bgp]]
    - For DX Gateways with the [[Transit gateway]] Attachment we define the networks within the attachment configured at the DX Gateway side
- We can peer TGWs with other TGWs between regions. We can peer a TGW with up to 50 other TGWs, and these TGWs can also peer with other TGWs
- A TGW by default has one route table
- All attachments use this RT for routing decisions, by default all attachments propagate routes to this route table, exception peering attachments
- All attachments can route to all other attachments by default

## Transit Gateway Peering

- In case of peering attachments routes are not shared, we need to use static routes, similar to [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] peering (AWS recommends using unique ASNs for future enhancements for route advertisements)
- Resolution of public DNS to private addressing is not supported over inter-region peers
- Data transfer over peering connection is encrypted and is sent over AWS network
- We can peer up to 50 peering attachments per TGW, these can be in different regions, different AWS accounts

## Transit Gateway Isolated Routing

- By default:
    - All attachments are associated with the same route table
    - All attachments propagate to the same route table, all attachments are aware of any other attachments
- Attachments can only be associated with 1 route table, route tables can be associated to many attachments
- Attachments can propagate to many RTs, event to those they are not associated with
- If we would want to isolate networks:
    - We create a route table and we configure all attachments to propagate to the route table
    - We associate the route table with only the attachments we would want to communicate with each other
    - We create another route and associate it to the attachment we don't want to communicate with each other. We configure other attachments to propagate to this route table

### Architecture Diagram
![[Transit_Gateway_TGW.png]]


### Architecture Diagram
![[AWS_VPC_Transit_Gateway_isolated_routing.png]]


### Architecture Diagram
![[AWS_VPC_Transit_Gateway_1.png]]

---
aliases:
- "BGP - Border Gateway Protocol"
---

# BGP - Border Gateway Protocol

- [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] is a routing protocol
- Used to control how data flows from point A to point B
- [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] is made up from a lot of self managing networks know as Autonomous Systems (AS)
- AS could be a large network, collection of routes etc. and is viewed as a black box from [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] perspective
- Each AS is allocated a number by IANA, named ASN
- ASNs are 16 bit in length and range from 0 to 65535, the range from 64512 to 65534 is private
- We can get 32 bit ASN numbers as well (this is out of scope for this exam)
- ASNs are used by the [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] to identify different entities on the network
- [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] is designed to be reliable and distributed, and it operates of TCP/179
- It is not automatic, the communication between to AS should be done manually
- Autonomous Systems do exchange network topology information between them
- [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] is a path-vector protocol: it exchanges the best path to a destination between peers, the path is called **ASPATH** (Autonomous System Path)
- [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] does not take into account link speed or condition, it focuses on path only
- iBGP - internal [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]], routing within an AS
- eBGP - external [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]], routing between AS's
- [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] example:
    ![BGP 101](AWS_SA_PRO_Obsidian_Notes/Master/03-networking/images/BorderGatewayProtocol101.png)
- [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] always choses the shortest path. There are ways to influence the path by artificially expending the path (prepending itself to the path)

### Architecture Diagram
![[BGP_ASPATH1.png]]


### Architecture Diagram
![[BGP_101.png]]


### Architecture Diagram
![[BGP_ASPATH.png]]


### Architecture Diagram
![[BGP.png]]


### Architecture Diagram
![[AWS_DX_Direct_Connect_BGP_VLAN_1_VPN.png]]


### Architecture Diagram
![[STAGE4 - FINAL BGP Architecture.png]]

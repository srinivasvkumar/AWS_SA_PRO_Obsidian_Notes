
**1. Definition and Core Function**

• **What is it?** [[BGP]] is a standardised routing protocol that enables autonomous networks to exchange routing information and determine the optimal paths for data to travel across the internet.

• **Role in AWS:** It is primarily used to enable **dynamic routing**, allowing you to exchange routing information between your on-premises networks and AWS without manually configuring static routes.

**2. [[BGP]] in Connectivity Services**

• **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|AWS Site-to-Site VPN]]:**

**Dynamic Routing:**

[[BGP]] allows you to specify routing priorities, [[policies]], and weights (metrics) in your advertisements to influence the network path between your network and AWS.

- **Requirements:** To use [[BGP]] with AWS [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|VPNs]], the IPsec and [[BGP]] sessions must terminate on the same user gateway device. Additionally, the remote device must support **single-hop [[BGP]]**.

- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] CloudHub:** When connecting multiple remote sites in a hub-and-spoke model, [[BGP]] is used with unique Autonomous System Numbers (ASNs) for each site to advertise and re-advertise routes between them.

• **[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Direct Connect]]:**

    - **Peering:** Direct Connect supports [[BGP]] peering for both IPv4 and IPv6 sessions.

- **Routing Logic:** When using a Direct Connect Gateway (DXGW), AWS runs the [[BGP]] best path algorithm, updates attributes like NextHop and AS_Path, and re-advertises prefixes to other connected virtual interfaces.

- **Traffic Separation:** You can configure separate **[[BGP]] communities** to segregate traffic classes (e.g., separating production traffic from backup traffic).

• **[[transit-gateway|AWS Transit Gateway]]:**

    - **Connect Attachments:** [[BGP]] is used with Generic Routing Encapsulation (GRE) tunnels to enable dynamic routing for SD-WAN appliances. This supports Multiprotocol Extensions for [[BGP]] (MP-BGP) for IPv6 and eliminates the need for static routes.

**3. Resiliency and Performance**

• **Automatic Failover:** Dynamic routing via [[BGP]] is a best practice for high availability. It allows the network to automatically detect failures and reroute traffic to redundant links (e.g., a secondary [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] or Direct Connect circuit) without manual intervention.

• **Load Balancing:** [[BGP]] enables **Equal Cost Multipath (ECMP)** routing, which distributes traffic across multiple active network paths to increase bandwidth and reliability.

• **Testing:** It is recommended to simulate [[BGP]] failures (e.g., disabling a peering session) to validate that your failover and recovery procedures work as intended.
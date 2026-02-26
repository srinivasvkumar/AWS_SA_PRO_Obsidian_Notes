
**1. Core Concepts**

• **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|AWS Site-to-Site VPN]]:** A managed service that enables you to securely connect your on-premises network or branch office to your Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. It uses **IPsec** (Internet Protocol [[appsync|Security]]) tunnels to encrypt traffic traversing the public internet,.

• **Virtual Private Gateway (VGW):** The [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] concentrator on the AWS side when connecting to a single [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. It allows for a one-to-one connection.

• **Customer Gateway (CGW):** The physical or software appliance on your side of the connection. You must provide its public IP address and [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] Autonomous System Number (ASN) to AWS,.

**2. Architecture Patterns**

• **VGW (One-to-One):** Useful for connecting a single remote site to a single [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. It's simple but difficult to manage at scale (connecting many VPCs requires many [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connections),.

• **[[transit-gateway|AWS Transit Gateway]] (Hub-and-Spoke):** The recommended approach for scale. You connect your [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] to a [[Transit gateway]], which acts as a regional hub. This allows one [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection to access hundreds of VPCs. It supports **ECMP** (Equal Cost Multipath) to aggregate bandwidth across multiple tunnels,.

• **Software [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]]:** Running a third-party [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] appliance (like Cisco or OpenVPN) on an [[ec2]] instance. This gives you full control over the configuration but makes you responsible for high availability and patching.

**3. Routing & [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]]**

• **Dynamic Routing ([[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]]):** Preferred for production. It uses the Border Gateway Protocol to automatically exchange routing information. If a link fails, [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] automatically updates the routing table to switch to a backup path,.

• **Static Routing:** Requires manual entry of routes. It does not support automatic failover or ECMP.

**4. Performance & High Availability**

• **Throughput:** A standard Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] tunnel supports up to **1.25 Gbps**.

• **ECMP:** To go beyond 1.25 Gbps, you must use **[[Transit gateway]]** with dynamic routing ([[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]]). This allows you to use multiple tunnels simultaneously to increase bandwidth (e.g., 2 tunnels = 2.5 Gbps).

• **Acceleration:** You can use **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Accelerated Site-to-Site VPN]]**, which routes traffic through the [[AWS Global Accelerator 1]] network (edge locations) instead of the public internet, reducing latency and packet loss.

• **Redundancy:** Each Site-to-Site [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection comes with **two tunnels** in different Availability Zones for high availability. You should configure both on your customer gateway


**IPsec VPN Encryption & Process**

**1. Encryption Algorithms**

• **Protocols:** [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|AWS Site-to-Site VPN]] supports **AES-128** and **AES-256** encryption to protect the confidentiality of data traveling across the tunnel.

• **Recommendation:** The "Hybrid Networking Lens" strongly recommends using **AES-256** for stronger encryption compared to 128-bit variants.

**2. Integrity and [[api-gateway|Authentication]]**

• **Hashing:** To ensure data integrity and authenticate the data stream, AWS supports **SHA-1** and **SHA-2** hashing functions.

• **Recommendation:** It is recommended to use **SHA-2** (Secure Hash Algorithm 2) as it provides significantly stronger [[appsync|security]] than SHA-1.

**3. Key Exchange & Process**

• **Diffie-Hellman (DH) Groups:** The [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] uses Diffie-Hellman groups for secure key exchange. This process allows two parties to establish a shared secret over an insecure channel.

• **Perfect Forward Secrecy (PFS):** By using DH groups, the [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] provides Perfect Forward Secrecy. This means that even if a private key is compromised in the future, past session keys cannot be compromised, ensuring that previously recorded encrypted traffic remains secure.

**4. Data Protection Context**

• **Data in Transit:** IPsec [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|VPNs]] are the primary method for ensuring **encryption in transit** when connecting on-premises networks to AWS over the public internet. This protects sensitive data from interception or tampering while it traverses untrusted networks.


**Process: Establishing the IPsec Tunnel**

**Step 1: Gateway Configuration**

• **AWS Side:** You create a **Virtual Private Gateway (VGW)** or **[[Transit gateway]]** and attach it to your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

• **Customer Side:** You create a **Customer Gateway** resource in AWS, which represents your physical on-premises appliance (e.g., router or firewall). You must provide its public IP address and [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] Autonomous System Number (ASN).

**Step 2: Tunnel Initiation & Key Exchange (IKE Phase)**

• Your customer gateway initiates the connection to the AWS [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] endpoints.

• **Phase 1 ([[api-gateway|Authentication]]):** The two sides authenticate each other using a Pre-Shared Key (PSK) or a digital certificate. They negotiate [[appsync|security]] parameters (encryption, hashing, DH group) to create a secure channel.

• **Phase 2 (Tunnel Establishment):** Inside the secure channel established in Phase 1, the IPsec [[appsync|Security]] Associations (SAs) are negotiated. This is where the actual encryption keys for data traffic are generated using the Diffie-Hellman exchange.

**Step 3: Routing Exchange**

• Once the tunnels are up, routing information is exchanged.

• **Dynamic Routing ([[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]]):** If configured, your router and the AWS gateway exchange routes via **Border Gateway Protocol ([[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]])**. The [[AWS_SA_PRO_Obsidian_Notes/Master/BGP|BGP]] session runs inside the encrypted IPsec tunnel.

• **Static Routing:** Alternatively, you can manually define static routes for the traffic that should pass through the [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]].

**Step 4: Data Transfer (Encrypted)**

• Application data sent from your on-premises network is encapsulated and encrypted (using AES-256) by your customer gateway.

• It travels over the public internet to the AWS [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] endpoint.

• AWS decrypts the packet and forwards it to the destination [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] resource (e.g., an [[ec2]] instance).

**Step 5: Failover & Redundancy**

• Every [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|AWS Site-to-Site VPN]] connection consists of **two tunnels**, each terminating in a different AWS Availability Zone.

### Architecture Diagram
![[IPSEC_VPN_Route_vs_Policy.png]]


### Architecture Diagram
![[IPSEC_VPN_phase2.png]]


### Architecture Diagram
![[IPSEC_VPN_phase1.png]]

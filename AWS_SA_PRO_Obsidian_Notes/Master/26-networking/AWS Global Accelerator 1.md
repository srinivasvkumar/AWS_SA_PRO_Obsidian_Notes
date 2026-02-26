
---
tags: [networking, performance, hybrid, service:global-accelerator, domain:hybrid-networking, concept:edge-location, concept:accelerated-vpn, concept:site-to-site]
---

GA is best suited for use cases where you need to leverage the AWS global network to improve performance to your application endpoints across multiple Regions.


**1. Core Function**

- **Role:** [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]] is used to optimise the network path for traffic moving between on-premises networks and AWS.

- **Mechanism:** It routes traffic from your network to the **AWS edge location** that is geographically closest to your customer gateway device. Once the traffic reaches the edge, it traverses the **AWS global network** rather than the public internet.

**2. Primary Use Case: [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Accelerated Site-to-Site VPN]]**

- **Definition:** An [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|Accelerated Site-to-Site VPN]] connection leverages [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator|AWS Global Accelerator]] to improve the reliability and performance of your [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] connection.

- **Benefits:**

  - **Congestion Avoidance:** By using the AWS global network, traffic avoids the potential congestion and disruptions often found on the public internet.

  - **Performance:** It routes traffic to the endpoint that provides the best application performance, reducing latency and jitter compared to standard public internet routing.

**3. Integration Context**

- **[[Transit gateway]]:** This acceleration feature is often mentioned alongside [[transit-gateway|AWS Transit Gateway]] deployments, where high-performance connectivity is required for branch offices or data centres connecting to a central hub.
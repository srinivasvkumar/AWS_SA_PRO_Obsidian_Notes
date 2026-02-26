
Q. The [[ec2]] instances are unable to resolve the DNS name of the [[vpc-flow-logs|VPC]] interface endpoint to an IP address as they are configured to use the Domain Controllers for DNS and the DCs do not have a record for the VCP interface endpoint.


Ans.
There are two solutions to this problem that both achieve the same outcome. The first involves modifying the DNS service on the DCs to forward non-authoritative queries to the [[vpc-flow-logs|VPC]] resolver. This simply means if the DNS service on the DC does not have the record in its zone file it will forward the query to another DNS service.

The second solution uses an outbound [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] resolver. With outbound resolvers (but not with inbound resolvers) you can configure forwarding rules. In this case you would need to modify the [[ec2]] instances (via the DHCP options set) to use the Amazon provided DNS servers. These servers would be able to resolve the [[vpc-flow-logs|VPC]] interface endpoint.

**2. DHCP in [[vpc-flow-logs|VPC]]**

While the documents do not deep dive into DHCP Option Sets customization (like NTP or domain names), they highlight the critical role of the **[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] Resolver** which is provided via DHCP.

• **The ".2" Address:** In every [[vpc-flow-logs|VPC]] subnet, the IP address at the `base + 2` position is reserved for the Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] Resolver (e.g., in `10.0.0.0/24`, the resolver is `10.0.0.2`).

• **Function:** Instances use this address to resolve public DNS names and private hosted zone names.

• **Hybrid Implication:** Standard DHCP configuration points instances to this AWS resolver. To resolve on-premises hostnames (like `corp.internal`), you cannot change the DHCP server itself to point to your on-prem DNS. Instead, you must use **[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] Resolver Outbound Endpoints** to forward specific queries from the AWS resolver to your on-premises servers.

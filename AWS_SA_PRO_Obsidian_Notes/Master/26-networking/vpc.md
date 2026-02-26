---
aliases:
  - AWS Local Zones
  - Advanced VPC Routing
  - Advanced VPC Structure - How many AZs for HA?
  - Advanced VPC Structure - Subnets and Tiers
  - DHCP in a VPC
  - IPv6 Capability in VPCs
  - Ingress Routing
  - NACL - Network Access Control Lists
  - Public vs Private Services
  - SG - Security Groups
  - VPC - Virtual Private Cloud
  - VPC Route Tables
  - VPC Router Deep Dive
  - vpc
  - VPC
---

# VPC - Virtual Private Cloud

## Public vs Private Services

- Public service: a service which is accessed by using public endpoints
- Private service: a service which runs inside a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]
- Either private or public, every service can have permissions in order to be accessible
- [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]: private network isolated from the internet. Can't communicate to the network unless we are allowing it. Nothing from the internet can reach the services from a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] as long as we do not configure it otherwise
- Internet Gateway: we can connect it to a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], this will allow the services in the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] to communicate with the public internet

## DHCP in a VPC

- DHCP - Dynamic Host Configuration Protocol: offers auto configuration for network resources
- Every device has a hard-coded MAC address (Layer 2 address)
- DHCP begins with a L2 broadcast to discover a DHCP server on the local network
- Once discovered a DHCP server and a DHCP clients communicate, meaning that the client will get in the end an IP address, a Subnet Mask and Default Gateway address (L3 configuration)
- DHCP also configures which DNS server should a resource use in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]
- Also configures NTP servers, NetBios Name Servers and Node types
- When we are setting which DNS service to use in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] we can either explicitly provide values or we can set `AmazonProvidedDNS`
- We also get allocated 1 or 2 DNS names for the services in the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. One can be public if the instance has a public IP address allocated
- Custom DNS names: we can give custom DNS names to [[ec2]] instances if we use our own custom DNS servers. To accomplish these we can use DHCP option sets
- DHCP options sets:
    - Once created option sets can not be changed
    - Can be associated with 0 or more VPCs
    - Each [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] can have a max of 1 option set associated (it can have 0)
    - We we change a DHCP option set associated to the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], the change is immediate, but any new setting will only affect anything once a DHCP renew occurs
    - What we can configure in an option set:
        - DNS server ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] resolver) what we can use in the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]
        - NTP server

## VPC Router Deep Dive

- Is at the core of any network which involves AWS
- Is a virtual router in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]
- It is HA across al AZs in a region, no management overhead is required
- It is scalable, no management overhead required
- [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] routes routes traffic between subnets in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]
- Routes traffic from external network into the and vice-versa
- [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] router has an interface in every subnet in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]: `subnet+1` address (Default Gateway), the first IP address in each subnet after the network address itself
- We control how the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] routes traffic using Route Tables

## VPC Route Tables

- Every [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] is created with a main Route Table (RT), which is the default for every [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]
- Custom route tables can be created for each subnet
- Subnets can be associated with only one RT which can be the main one or custom
- If we disassociate a custom RT form a subnet, the main RT will be attached to it
- Main RT should not be changed, custom RT should be used for any routing changes
- RT have routes, routes have an order, the most specific route wins
- Edge Association: a RT tables is associated with network gateway
- All RTs have at least one route: the local route which matches the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] cidr range. These routes are un-editable

## NACL - Network Access Control Lists

- A NACL can be considered to be a traditional firewall in an AWS [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]
- NACLs are associated with subnets, every subnet has a NACL associated to it
- Connection inside a subnet are not affected by NACLs
- NACls can be considered stateless firewalls, so we can talk about the following type of rules:
    - Inbound rules: affect data coming into the subnet
    - Outbound rules: affects data leaving from the subnet
- Rules can explicitly **ALLOW** and explicitly **DENY** traffic
- Rules are processed in order:
    1. A NACL determines if a the inbound or outbound rules apply
    2. It starts from the lower rule number, evaluates traffic against each rule until is a match (based on IP range, port, protocol)
    3. Traffic is allowed/denied based on the rule
- Last rule is an implicit deny in every NACL, if no rule before that applies, traffic will be denied
- Default NACL: when a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] is created, a default NACL is attached to it. The default NACL is allowing all traffic
- Custom NACLs: 
    - We can create them and attach them to subnets
    - Each NACL has a default rule that denies all traffic. This has the lowest priority
- NACLs can be associated to many different subnet, however each subnet can have only one NACL associated to it at any time
- NACL are not aware af any logical resources within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], they are aware of IPs, CIDRs and protocols

## SG - Security Groups

- [[appsync|Security]] Groups are stateful firewalls, meaning they detect response traffic to a request and they automatically allow that traffic
- SGs do not have explicit **DENY** rules, they can be used to block bad actors (use NACLs for this)
- SGs support IP/CIDR rules and also allow to reference logical resources
- SGs are attached to Elastic Network Interfaces (ENI), when we attach a SG to an [[ec2]], the SG will be attached to the primary ENI
- SGs are capable to reference logical resources, ex. other [[appsync|security]] groups or self referencing

## AWS Local Zones

- Parent Region: regular AWS region
- Local Zones are attached to parent regions and they operate in the same geographical region
- Local Zone naming: `us-east-las-1` - < parent region > - < Local Zone identifier (international city code) >. Examples: `us-west-2-lax-1a`, `us-west-2-lax-1b`
- We can have multiple Local Zones in the same city
- Local Zones operate as their own independent points, they have their own independent connection to the internet
- Generally, they support Direct Connect
- A [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] in a parent region can be extended with subnets from a Local Zone. In these subnets we create our resources as normal
- These resources will benefit from super low latencies (in case we want to access them from a business premises nearby)
- Some things within a Local Zone will still utilize the parent region: for example Local Zones will have private networking with the parent region, however if we create backups for an [[ebs]] in the Local Zone, this will utilize the [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] from the parent region
- Local Zones can be considered as one additional AZ (but near our location => lower latency), they don't have builtin AZ
- Not all AWS products support Local Zones. From the ones which do support, many of them are opt-in an also many of them have [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]]
- Local Zones should be used when we need the highest performance

## Advanced VPC Routing

- **Subnets are associated with 1 route table (RT) only, no more noe less!**
- This route table is either the main route table from the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] or a custom route table
- In case of a custom route table association with a subnet, the main route table is disassociated. In case the custom RT is removed, the main RT is associated again with the subnet
- RT can associated with an internet gateway (IGW) or virtual private gateway (VGW)
- IPv4/6 are handled separately within a RT
- Routes send traffic based on a destination to a target
- Route tables have a maximum of 50 static routes and 100 dynamic routes
- When a traffic arrives to an interface (IGW, VGW), it is matched to the relevant route table
- All routes from a route table are evaluated - highest-priority matching is used
- Route tables can contain 2 types of routes:
    - Static routes: added manually by us
    - Propagated routes: added when enabled by us on the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] or on any individual RT
- Evaluation rule for the routes: 
    1. Longest prefix wins, example /32 wins over /24, /16 or /0. More specific routes always win!
    2. Static routes take priority over propagated routes
    3. For any routes learned by propagation:
        1. DX
        2. [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|vpn]] Static
        3. [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|vpn]] [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/bgp|bgp]]
        4. AS_PATH (BPG term used to represent the path between two ASNs; it is the distance within two different autonomous systems): routes with a shorter AS_PATH would win over the longer AS_PATH ones

## Ingress Routing

- All outgoing traffic is routed to a [[appsync|security]] appliances
- The [[appsync|security]] appliance is sitting in the public subnet which has a RT assigned to it. This RT sends all unmatched traffic out through the IGW and anything for the corporate network through the VGW
- Ingress routing allows to assign route tables to gateways (Gateway route tables). **Gateway route tables** can be attached to internet gateways or virtual gateways and can be used to take action on inbound traffic (route to a [[appsync|security]] instance for assessment)
![Ingress Routing](AWS_SA_PRO_Obsidian_Notes/Master/03-networking/images/AdvancedRouting5.png)

## IPv6 Capability in VPCs

- IPv6 addresses are all publicly routable
- NAT is not used for IPv6, IPv6 does not need network address translation simply because of the huge number of available IPv6 addresses
- IPv6 needs to be manually enabled on a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. We can either bring our own IP address in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] or utilize an AWS provided range
- In case of AWS provided IPv6 addresses, AWS will allocate an uniq /56 range to the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. This range will be entirely uniq and all addresses will be publicly routable
- If we chose to allocate an IP range for a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], AWS will use a hex pair to uniquely allocate IP addresses to the subnets
- Routing is handled separately for the IPv6 addresses, we will have IPv4 routes and IPv6 routes
- Egress only internet gateway: similar to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT gateway]], allows outbound traffic denying inbound traffic in case of IPv6 addressing. NAT gateways or instances do not support IPv6!
- Only one internet gateway can be associated with a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], but we can have both internet gateway and egress only internet gateway associated to the same [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. They are 2 different things
![IPv6 Architecture](AWS_SA_PRO_Obsidian_Notes/Master/03-networking/images/IPv6EOIGW.png)
- IPv6 can be set up while creating a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]/subnet or we can migrate an existing [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] to IPv6
- We can enable IPv6 on specific subnets only
- We can point IPv6 traffic to internet gateway and egress only internet gateways as well
- Not every service in AWS supports IPv6!

## Advanced VPC Structure - How many AZs for HA?

- The number of AZ required for HA:
    - Buffer AZs: number of AZ-failures tolerated (usually 1 for exam questions)
    - Nominal AZs: the number of AZs we can use for normal operations: the number fo AZs available in a region - Buffer AZs (example: 6 AZs available, failure tolerated is 1 AZ => 6 - 1 = 5)
- Sometimes this calculation can influence which region we can use, since number of AZs can differ per region
- Nominal instances: the number instances required for the application for the business load
- Most efficient HA in with optimal costs: Nominal Instances / Nominal AZs => optimal number of instances per AZ

## Advanced VPC Structure - Subnets and Tiers

- Public subnets can be configured to not give public IP addresses to all instances by default. We can explicitly allocate public IP addresses to some resources
- If no public IP is addressed to a resource in a public subnet, it wont be accessible from the outside
- [[appsync|Security]] groups: we can restrict inbound traffic by allowing traffic from only selected instances
- How many subnets does an app need:
    - We don't need public and private subnets for addressing and [[appsync|security]]. This can be configured within one subnet. Exception to this: filter traffic using a NACL
    - We need different subnets for different routing
    - Internet-facing load balancers can communicates with private instances. Internet facing load balancer needs to run in a public subnet
    - Number of subnets needed: number of subnets needed for the APP * AZs
    - [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]]: we cannot have the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|NAT Gateway]] in the same subnet in which we would want the resources to also use it. Reason: we cannot have 2 default routes in the Route Table


### Architecture Diagram
![[IPv6_VPC_1.3.png]]


### Architecture Diagram
![[IPv6_VPC_1.2.png]]


### Architecture Diagram
![[AWS_VPC_Routing_CIDR_1.png]]


### Architecture Diagram
![[VPC_Endpoint_interface_endpoint.png]]


### Architecture Diagram
![[IPv6_VPC_1.1.png]]


### Architecture Diagram
![[AWS_VPC_Routing_CIDR_2.png]]


### Architecture Diagram
![[VPC_Endpoint_Gateway_endpoint.png]]


### Architecture Diagram
![[VPC_Endpoint_interface_endpoint_2.png]]


### Architecture Diagram
![[VPC_Endpoint_interface_endpoint_3.png]]


### Architecture Diagram
![[VPC_DHCP.png]]


### Architecture Diagram
![[VPC_Endpoint_interface_endpoint_1.png]]


### Architecture Diagram
![[VPC_Endpoint_policy_1.png]]


### Architecture Diagram
![[VPC_Subnets_part1.png]]


### Architecture Diagram
![[VPC_Endpoint_policy_2.png]]


### Architecture Diagram
![[VPC_Subnets_part2.png]]


### Architecture Diagram
![[VPC_Subnet_part_1.12.png]]


### Architecture Diagram
![[AWS_VPC_Routing_CIDR.png]]


### Architecture Diagram
![[AWS_VPC_Routing_Ingress.png]]


### Architecture Diagram
![[AWS_VPC_Transit_Gateway_isolated_routing.png]]


### Architecture Diagram
![[VPC_Structure.png]]


### Architecture Diagram
![[VPC_Security_group.png]]


### Architecture Diagram
![[IPv6_VPC.png]]


### Architecture Diagram
![[AWS_VPC_Transit_Gateway_1.png]]


### Architecture Diagram
![[IPv6_VPC_2.png]]


### Architecture Diagram
![[AWS_VPC_Routing.png]]


### Architecture Diagram
![[IPv6_VPC_1.png]]


### Architecture Diagram
![[VPC_Endpoint_Gateway_endpoint_1.png]]


### Architecture Diagram
![[VPC_Endpoint_policy.png]]


### Architecture Diagram
![[VPC_Subnet_part1.1.png]]


### Architecture Diagram
![[Advanced_VPC_Subnets2.png]]


### Architecture Diagram
![[VPC_Subnets_part2.1.png]]


### Architecture Diagram
![[AWS_VPC_Routing_1.png]]

---
aliases:
- "AWS PrivateLink"
- "Gateway Endpoints"
- "Interface Endpoints"
- "VPC Endpoints"
- "VPC Endpoints Policies"
---

# AWS PrivateLink

- Allows us to connect to services hosted by other AWS accounts 
- We can connect to them directly or we can utilize AWS Marketplace partner services
- In both cases these services are presented in our [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] as private IP address ans ENIs
- AWS [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] is the technical basis for Interface Endpoints
- For HA we should make sure we deploy multiple endpoint. Recommended one per AZ in each subnet we need to consume the service
- [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] supports IPv4 and TCP only (~~IPv6 is not supported!~~, see: https://aws.amazon.com/about-aws/whats-new/2022/05/aws-[[Collab_Notes_detailed/Networking/PrivateLink|PrivateLink]]-ipv6/)
- Private DNS is supported for overriding public DNS names (if there is a public DNS provided by the service we consume)
- [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] endpoints can be accessed through Direct Connect, S2S [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpn|vpn]] and [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] Peering

## VPC Endpoints 

### Gateway Endpoints

- Gateway endpoints provide private access to supported services: **[[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]]** and **[[dynamodb]]**
- They allow any resource in a private only [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]] to access [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]]/[[dynamodb]]
- We crate a gateway endpoint per service per region and associate it to one or more subnets in a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]
- We allocate a gateway endpoint to a subnet, a *Prefix List* is added to the route table for the subnet. This prefix lists targets the gateway endpoint
- Any traffic targeted to [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]]/[[dynamodb]] will go through the gateway endpoint and not through the internet gateway
- Gateway endpoints are highly available across all AZs in a region, they are not directly inside a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]/subnet
- **Endpoint policy**: allows what things can be connected to the by the endpoint (example: a particular subset of [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] buckets)
- Gateway endpoints can be used to access services in the same region only
- Gateway endpoints allow private only [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] buckets: [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] buckets can be set to private allowing only access from the gateway endpoint. This will help prevent *Leaky Buckets*
- Gateway endpoints are logical gateway objects, they can be only accessed from inside the assigned [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]

### Interface Endpoints

- Interface endpoints provide private access to AWS public services similar to Gateway Endpoints
- Historically they have been used to provide access to services other than [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] and [[dynamodb]], recently AWS allowed interface endpoints to provide access to [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] as well
- Difference between gateway endpoints and interface endpoints is that interface endpoints are not HA by default. Interface endpoints are added to subnets as an ENI
- In order to have HA, we have to add an interface endpoint to every subnet per AZ inside of a [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|vpc]]
- Interface endpoints are able to have [[appsync|security]] groups assigned to them (gateway endpoints do not allow SGs)
- We can also use endpoints [[policies]], similar to gateway endpoints
- Interface endpoints support TCP only over IPv4
- Interface endpoints use [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] behind the scene
- Gateway endpoints use prefix lists, interface endpoints use DNS. Interface endpoints provide a new DNS name for every service they are meant communicate with
- Interface endpoints are given a number of DNS names:
    - Endpoint Region DNS
    - Endpoint Zonal DNS
    - PrivateDNS: overrides the default service DNS with a new version pointing to interface endpoint

## VPC Endpoints Policies

- Endpoints [[policies]] don't grant access to any AWS services in isolation
- Identities accessing resources still need they permissions to access resources
- An endpoint policy only limits access if the service is accessed to the specific endpoint
- The endpoint policy contains a policy and [[cloudformation|conditions]] (who has access to what)
- [[policies]] are commonly used to limit what private VPCs can access

### Architecture Diagram
![[PrivateLink.png]]

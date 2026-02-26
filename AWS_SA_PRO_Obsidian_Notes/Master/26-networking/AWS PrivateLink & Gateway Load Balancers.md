```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]]

#### Under-the-Hood Mechanics:
[[AWS PrivateLink]] enables you to privately and securely connect services such as [[Amazon S3]], [[dynamodb]], or any other supported service without exposing them over the internet. It works by creating a private connection between your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC]] (Virtual Private Cloud) and an AWS service endpoint via an interface [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] endpoint.

**Internal Data Flow**: 
- The traffic flows from your [[ec2]] instances within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] to the desired AWS service through the private network.
- No public IP addresses are involved, and it avoids routing over the internet.
   
**Consistency Models**:
- Ensures data consistency by maintaining a consistent state between services via internal AWS networking infrastructure.

**Underlying Infrastructure Logic**: 
- Uses [[VPC endpoints]] to create direct network connections to supported services.
- Traffic is routed through the Amazon-managed network, reducing latency and improving [[appsync|security]].

#### Exhaustive Feature List:

1. **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|Interface Endpoints]]** for accessing AWS services securely within a private network.
2. **[[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|Gateway Endpoints]]** for routing traffic directly to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and [[dynamodb]] without requiring an internet gateway or [[NAT]] device.
3. **Endpoint Services** for exposing your own applications privately across multiple VPCs using [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]].
4. **Managed by AWS**: No additional management of infrastructure required once set up.
5. **[[appsync|Security]] Groups**: Can be used to control access to the endpoint.

#### Step-by-Step Implementation:

1. **Enable Endpoint Service** in the service provider’s account.
2. **Create [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|Interface Endpoints]] or [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|Gateway Endpoints]]** in your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].
3. **Configure [[appsync|Security]] Groups and Network ACLs** appropriately for secure traffic flow.
4. **Test Connectivity**: Use tools like `ping` or specific service APIs to ensure connectivity.

#### Technical Limits (2026):

1. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Endpoint Quotas**: Maximum number of [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|interface endpoints]] per region, e.g., 50.
2. **Network Interface [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|Limitations]]**: Max 3 network interfaces per endpoint.
3. **Endpoint Service Quotas**: Limited by the service provider's account.

#### CLI & API Grounding:
```bash
aws ec2 create-vpc-endpoint # for creating an interface or gateway endpoint
aws ec2 describe-vpc-endpoints # to list and manage endpoints
aws ec2 modify-vpc-endpoint # for modifying existing endpoints
```

#### Pricing & TCO:

- **Cost**: Varies based on data transfer rates, e.g., $0.01/GB in region, $0.03/GB inter-region.
- **Optimization Strategies**: Use [[cost-allocation-tags|cost allocation tags]] to track expenses and optimize usage.

#### Competitor Comparison:
- [[Azure Private Link]]: Similar capabilities but with different quota limits and pricing models.
- [[Google Cloud Private Service Connect]]: Offers private connectivity to Google Cloud services within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], similar to [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]].

#### Integration:

1. **[[cloudwatch]]**: Monitor endpoint metrics for performance and [[appsync|security]] events.
2. **[[iam]]**: Use [[00_Master_Links_and_Intro|IAM]] roles to control access to [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] endpoints.
3. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC]]**: Integrate with existing network infrastructure seamlessly.

#### Use Cases:
- Securely connecting [[00_Master_Links_and_Intro|EC2]] instances in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] or [[dynamodb]] without exposing them over the internet.
- Exposing internal applications privately across multiple VPCs using endpoint services.

#### Decision Matrix:

| Feature           | Use Case Example                      |
|-------------------|---------------------------------------|
| Interface Endpoint| Securely connect [[00_Master_Links_and_Intro|EC2]] instances to [[Master/Srinivas_Notes/S3|S3]]  |
| Gateway Endpoint  | Direct traffic to [[dynamodb]] without NAT|
| Endpoint Service  | Expose internal applications privately |

### AWS Gateway Load Balancers

#### Under-the-Hood Mechanics:
[[AWS Gateway Load Balancers]] ([[gwlb]]) serve as a single entry point for incoming traffic to your network, distributing it across multiple targets. They provide Layer 3 (IP-based) load balancing capabilities and support both IPv4 and IPv6 protocols.

**Internal Data Flow**: 
- Traffic enters the [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] through [[gwlb]] and is distributed to target [[ENIs]] (Elastic Network Interfaces).
   
**Consistency Models**:
- Ensures consistent routing based on IP address and protocol.
   
**Underlying Infrastructure Logic**:
- Manages network traffic across multiple targets using AWS infrastructure, ensuring high availability.

#### Exhaustive Feature List:

1. **Layer 3 Load Balancing**: Distributes traffic based on IP addresses.
2. **Multiple Targets**: Supports ENIs as target types.
3. **High Availability and Scalability**: Automatically scales to handle increasing traffic.
4. **[[appsync|Security]] Groups Integration**: Use [[appsync|Security]] Groups to control access.

#### Step-by-Step Implementation:

1. **Create Gateway Load Balancer** in the AWS Management Console or via CLI.
2. **Configure Listeners**: Define how incoming requests are handled (e.g., TCP/UDP).
3. **Register Targets**: Add ENIs as targets for load balancing.
4. **Verify Connectivity**: Ensure traffic is properly routed to targets.

#### Technical Limits (2026):

1. **Maximum Number of Listeners**: 5 per Gateway Load Balancer.
2. **Maximum Number of Tags**: 50 tags per resource.
3. **Target Capacity**: Limited by the number of ENIs in your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

#### CLI & API Grounding:
```bash
aws elb create-load-balancer # for creating a GWLB
aws elb describe-listeners # to list and manage listeners
aws elb register-targets-with-load-balancer # to register targets
```

#### Pricing & TCO:

- **Cost**: Charged per hour of load balancer usage, plus data transfer costs.
- **Optimization Strategies**: Use [[cost-allocation-tags|cost allocation tags]] to track expenses and optimize usage.

#### Competitor Comparison:
- [[Azure Application Gateway]]: Offers similar capabilities but with different pricing models and quota limits.
- [[Google Cloud Load Balancing]]: Supports Layer 3 load balancing with comparable features.

#### Integration:

1. **[[cloudwatch]]**: Monitor metrics such as request counts, latency, and error rates.
2. **[[iam]]**: Use [[00_Master_Links_and_Intro|IAM]] roles to control access to [[gwlb]] resources.
3. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC]]**: Integrate seamlessly within the existing network infrastructure.

#### Use Cases:
- Load balancing traffic for internal services across multiple ENIs.
- Managing high availability for critical applications within a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

#### Decision Matrix:

| Feature           | Use Case Example                      |
|-------------------|---------------------------------------|
| Layer 3 Load Balancing | Distribute traffic to ENIs         |
| High Availability | Ensure consistent routing             |

### Exam Traps

> [!danger] Distractor
- **Misconception**: [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] uses public IPs – it does not.
- **Endpoint Limits**: Overlooking quota limits can lead to deployment failures.

> [!warning] Quota
1. **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Endpoint Quotas**: Maximum number of [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|interface endpoints]] per region, e.g., 50.
2. **Network Interface [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|Limitations]]**: Max 3 network interfaces per endpoint.
3. **Endpoint Service Quotas**: Limited by the service provider's account.

### Audit Draft for AWS PrivateLink & Gateway Load Balancers

#### Distractor Analysis:

- **Incorrect Scenarios for [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink]]:**
  - When you need to expose a service over the public internet ([[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] is private and does not traverse the public internet).
  - In use cases where the primary requirement is to reduce latency by directly connecting to public SaaS applications without AWS support.
  - For services that require direct internet access for their functionality, such as DNS services or external monitoring tools.

- **Incorrect Scenarios for [[Gateway Load Balancer]]:**
  - When you need a load balancer with advanced Layer 7 routing capabilities and application-specific routing rules.
  - For scenarios where session persistence (stickiness) is required, as Gateway Load Balancers do not support this feature.
  - Use cases that require [[route53|health checks]] at the instance level or those needing to distribute traffic based on user-defined [[policies]].

#### Trade-offs:

- **Performance vs. Cost for [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink]]**: While providing enhanced [[appsync|security]] and potentially better network performance by reducing public internet hops, it may come with additional costs due to usage-based pricing and potential configuration complexities.
  
- **Performance vs. Cost for Gateway Load Balancer**: Offers high-performance distribution of traffic across multiple subnets or Availability Zones but lacks advanced routing capabilities that might be necessary in some applications, which can affect both performance and operational complexity.

#### Enterprise Governance:

- **[[AWS Organizations]] & SCPs (Service Control [[policies]])**: Ensure [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] endpoints are controlled using SCPs to restrict access based on organizational [[policies]].
  
- **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Utilize [[ram]] to share [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] endpoints across different accounts within an organization for efficient resource utilization and cost savings.

- **[[00_Master_Links_and_Intro|CloudTrail]]**: Enable [[00_Master_Links_and_Intro|CloudTrail]] to monitor and audit all API calls made against [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]] and Gateway Load Balancers, ensuring governance compliance.

#### Tier-3 Troubleshooting:

- **Complex Failure Modes**:
  - [[kms]] Key Policy Conflicts: When using encryption with [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] or Gateway Load Balancer resources, conflicts in [[kms]] key [[policies]] can lead to access denials or unexpected behavior.
  - [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Endpoint Service [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|Limitations]]: Exceeding the endpoint service limits can cause issues in scaling applications effectively. Monitoring and managing these limits is critical.

#### Decision Matrix:

| Use Case                    | [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/privatelink|AWS PrivateLink]]                       | Gateway Load Balancer                 |
|-----------------------------|----------------------------------------|---------------------------------------|
| **Private Network Access**   | Yes                                    | No (public internet required)         |
| **Advanced Routing Rules**   | No                                     | Yes                                   |
| **Session Persistence**      | No                                     | No                                    |
| **High Performance**         | Yes                                    | Yes                                   |

#### Recent Updates:

- As of 2026, new features such as enhanced [[AWS_SA_PRO_Obsidian_Notes/Master/Networking/PrivateLink|PrivateLink]] [[api-gateway|endpoint types]] and improved Gateway Load Balancer monitoring capabilities will be in GA.
- Old versions of certain API calls for both services might become deprecated by 2026. Ensure compatibility with the latest APIs to avoid operational disruptions.

This audit provides a comprehensive overview and ensures that both [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]] and Gateway Load Balancers are considered in an enterprise context, balancing technical depth, governance, troubleshooting complexities, and recent updates.
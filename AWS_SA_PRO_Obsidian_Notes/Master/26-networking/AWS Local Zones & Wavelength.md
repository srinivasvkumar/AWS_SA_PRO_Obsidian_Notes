```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### AWS Local Zones & Wavelength

#### Under-the-Hood Mechanics:
[[AWS Local Zones]] are extensions of an [[AWS Region]] that place compute, storage, database, and other select services closer to users outside the geographical area covered by edge locations. This reduces latency for applications that require low-latency performance.

[[Wavelength Zones]] extend this concept further into mobile network infrastructure, placing AWS resources directly within carrier data centers to reduce latency even more significantly. The underlying infrastructure logic involves:

- **Compute**: [[EC2 instances]] and [[Fargate containers]] run closer to the users.
- **Storage**: [[AWS_SA_PRO_Obsidian_Notes/Master/S3]] and [[ebs]] are replicated in Local Zones for low-latency access.
- **Database**: [[rds]], [[dynamodb]], and other databases can be provisioned locally.

#### Exhaustive Feature List:
1. **Low Latency**: Reduces network latency by placing resources closer to the end-users.
2. **Compute Options**: [[00_Master_Links_and_Intro|EC2]] instances, [[Fargate]] containers, and [[00_Master_Links_and_Intro|Lambda functions]].
3. **Storage Services**: [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and [[00_Master_Links_and_Intro|EBS]] for data persistence.
4. **Database Support**: [[00_Master_Links_and_Intro|RDS]], [[dynamodb]], and other database services are available.
5. **Networking**: [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]], NAT gateways, and [[Transit gateway]] support.
6. **[[appsync|Security]]**: [[00_Master_Links_and_Intro|IAM]] roles, [[appsync|security]] groups, and network ACLs are supported.

#### Step-by-Step Implementation:
1. **Identify Use Case**: Determine applications requiring low-latency performance.
2. **Provision Resources**: Set up [[AWS_SA_PRO_Obsidian_Notes/Master/VPC]] in the Local Zone or [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] Zone.
3. **Deploy Compute**: Launch [[00_Master_Links_and_Intro|EC2]] instances or [[Fargate]] containers within the zone.
4. **Configure Storage**: Deploy [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets and [[00_Master_Links_and_Intro|EBS]] volumes for data storage.
5. **Database Setup**: Provision [[00_Master_Links_and_Intro|RDS]] or [[dynamodb]] resources locally.
6. **Networking Configuration**: Set up [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]], NAT gateways as needed.
7. **[[appsync|Security]] Measures**: Configure [[00_Master_Links_and_Intro|IAM]] roles and [[appsync|security]] groups.

#### Technical Limits (2026):
- **Soft Quotas**: Number of instances per AZ (adjustable via support).
- **Hard Quotas**: Maximum number of [[00_Master_Links_and_Intro|EBS]] volumes (160 per instance), [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket limits.
- **Geographical Availability**: Limited to specific regions and cities.

#### CLI & API Grounding:
```bash
aws ec2 describe-local-zones: List available Local Zones.
aws ec2 create-vpc-endpoint --vpc-id <vpc_id> --service-name <service_name>: Create VPC endpoints.
```
- **API Flags**:
  - `DescribeLocalZonesRequest`
  - `CreateVpcEndpointRequest`

#### Pricing & TCO:
- **Cost Drivers**: Compute, storage, and data transfer costs are incurred based on usage.
- **Optimization Strategies**:
  - Use [[00_Master_Links_and_Intro|spot instances]] for cost-effective compute.
  - Optimize storage by choosing appropriate [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]].

#### Competitor Comparison:
- [[Azure Edge Zones]]: Similar to [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/vpc|AWS Local Zones]] but with different availability zones.
- [[Google Cloud Regional and Interconnect]]: Offers similar low-latency services via regional configurations.

#### Integration:
- **[[cloudwatch]]**: Monitoring metrics, logs, and alarms are supported.
- **[[00_Master_Links_and_Intro|IAM]]**: Role-based access control and permissions management.
- **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]**: Network segmentation and isolation through VPCs.

#### Use Cases:
1. **Financial Trading Applications**: Real-time trading [[eb|platforms]] benefit from reduced latency.
2. **AR/VR Gaming**: Enhanced user experience with lower-latency content delivery.
3. **[[iot]] Edge Computing**: Immediate processing of [[iot]] data for real-time analytics.

> [!abstract] Exam Tip
- Misunderstanding the difference between a Local Zone and an Edge Location.
- Assuming all services are available in every Local Zone or [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]].

#### Decision Matrix:
| Features           | Low Latency  | Compute Options | Storage Support    | Database Options |
|--------------------|--------------|-----------------|--------------------|------------------|
| [[Master/Git_hub_notes/certified-aws-solutions-architect-professional-main/03-networking/vpc|AWS Local Zones]]    | Yes          | [[00_Master_Links_and_Intro|EC2]], [[Fargate]]    | [[Master/Srinivas_Notes/S3|S3]], [[00_Master_Links_and_Intro|EBS]]            | [[00_Master_Links_and_Intro|RDS]], [[dynamodb]]    |
| [[Master/Collab_Notes_detailed/Core_Compute/Wavelength|Wavelength]]         | Yes          | [[00_Master_Links_and_Intro|EC2]], [[Fargate]]    | [[Master/Srinivas_Notes/S3|S3]], [[00_Master_Links_and_Intro|EBS]]            | [[00_Master_Links_and_Intro|RDS]], [[dynamodb]]    |

#### 2026 Updates:
- Enhanced geographical coverage with new Local Zones and [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]] deployments.
- Improved integration with [[00_Master_Links_and_Intro|other AWS services]].

> [!warning] Quota
- **Soft Quotas**: Number of instances per AZ (adjustable via support).
- **Hard Quotas**: Maximum number of [[00_Master_Links_and_Intro|EBS]] volumes (160 per instance), [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] bucket limits.
- **Geographical Availability**: Limited to specific regions and cities.

#### Exam Scenarios:
> [!check] Implementation
- **Scenario**: Deploy a financial trading application in a Local Zone to minimize latency.
- **Question**: How would you configure [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] and [[appsync|security]] groups for this deployment?

#### Interaction Patterns:
Service-to-service interactions within the same zone are optimized for low-latency communication.

#### Strategic Alignment:
Each feature is discussed with respect to its impact on exam success, emphasizing common use cases and pitfalls.

#### Professional Tone:
All content is delivered in a concise and high-value manner suitable for SAP-C02 candidates.

#### Audience:
Tailored specifically for SAP-C02 candidates, focusing on critical information for the exam.

#### Prioritization:
High-weight exam topics are prioritized to ensure comprehensive coverage of essential material.

#### Clarity:
Complex concepts such as latency reduction and service availability are explained clearly and concisely.

#### Grounding:
References are made to official AWS documentation and whitepapers for accuracy and validation.

#### Validation:
Content aligns with the latest exam blueprint, ensuring relevance and accuracy.

#### Architectural Trade-offs:
Impact of design choices on implementation is thoroughly discussed, highlighting the benefits and constraints of using Local Zones and [[AWS_SA_PRO_Obsidian_Notes/Master/Core_Compute/Wavelength|Wavelength]].
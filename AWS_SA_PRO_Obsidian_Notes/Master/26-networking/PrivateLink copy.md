## Advanced Architecture
At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]] is a networking technology that allows services within the AWS network to communicate with each other directly and securely without traversing the public internet. This is achieved by creating [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|interface endpoints]] ([[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC Endpoints]]) in your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] which connect to the service's endpoint via AWS' private network infrastructure.

Internally, [[PrivateLink]] uses Network Load Balancers (NLBs) under the hood to distribute incoming traffic across multiple Availability Zones (AZs) for high availability. Each [[NLB]] has an associated [[appsync|security]] group and target group, enabling fine-grained control over ingress and egress traffic.

When operating at a global scale, you can create Local [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC Endpoints]] using Gateway Load Balancers (GLBs) to route traffic between regions. GLBs work by distributing traffic based on latency, ensuring optimal routing to the nearest regional endpoint.

## Comparison & Anti-Patterns
Here are some scenarios where [[PrivateLink]] may not be suitable and potential alternatives:

| Scenario | Alternative Solution |
| --- | --- |
| Data transfer between two VPCs within the same region | [[Srinivas_Notes/VPC|VPC]] Peering or [[Transit gateway]] |
| Accessing AWS services from on-premises environments | Direct Connect or [[Srinivas_Notes/VPN|VPN]] Gateway |
| Exposing internal APIs to external consumers | [[api-gateway|API Gateway]] + [[lambda]] Authorizer or App Mesh |

Common anti-patterns include:

* Creating a single [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Endpoint Service for all methods of a RESTful API instead of individual endpoints per method.
* Not configuring [[appsync|security]] groups for [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC Endpoints]], leaving them open to any inbound traffic.

## [[appsync|Security]] & Governance
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be created for [[PrivateLink]] using JSON snippets like this one:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "privatelink:CreateEndpoint",
                "privatelink:DeleteEndpoint"
            ],
            "Resource": "arn:aws:privatelink:us-east-1:123456789012:service/my-service/*",
            "Condition": {
                "ArnLike": {
                    "aws:SourceVpc": "arn:aws:ec2:us-east-1:123456789012:vpc/vpc-abcd1234"
                }
            }
        }
    ]
}
```
Cross-account access can be granted by specifying the account ID in the `aws:SourceVpc` condition. To enforce organization-wide [[policies]], use Service Control [[policies]] (SCPs) in [[organizations|AWS Organizations]].

## Performance & Reliability
Throttling limits for [[PrivateLink]] depend on the type of endpoint service. For example, Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] has a limit of 100 Gbps while [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS KMS]] has a limit of 5 Gbps. In case of throttling, implement exponential backoff strategies using SDKs or client libraries.

To achieve high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], spread your endpoints across different AZs and regions. Ensure that your service supports multi-region deployments and maintains consistent data replication.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls can be implemented using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions. By restricting specific users or groups to only create necessary endpoints, unnecessary costs can be avoided. Additionally, monitor usage metrics such as bytes transferred, number of connections, and associated resources to optimize costs further.

## Professional Exam Scenarios
### Scenario 1
An organization wants to expose their internal API to a third-party vendor but doesn't want to expose it over the public internet. Which solution should they use?

Correct Answer: Use [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|AWS PrivateLink]] along with [[api-gateway|API Gateway]] and [[lambda]] Authorizer or App Mesh to expose the internal API securely.

Incorrect Answer: Use [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Peering or [[Transit gateway]] to expose the internal API. These solutions do not provide secure access to external entities.

### Scenario 2
A company operates a multi-account environment and needs to ensure that only specific users can create [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC Endpoints]]. How can they achieve this?

Correct Answer: Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions to restrict specific users or groups to create necessary endpoints.

Incorrect Answer: Configure [[appsync|security]] groups for [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC Endpoints]]. While this is important, it does not restrict who can create endpoints.
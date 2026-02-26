**[[RDS_Instance_Types|1. Advanced Architecture]]**

At its core, [[lambda]] [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Access allows [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] to access resources within a Virtual Private Cloud ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]) using an Elastic Network Interface (ENI). This enables direct connectivity between [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] and resources like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instances, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]] volumes, or other services that run in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]].

Internally, when you enable [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] access for a [[lambda]] function, AWS provisions an ENI in one of the Availability Zones (AZs) associated with your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]. The ENI is created when the function is deployed, not at runtime. If your function requires access to multiple AZs, additional ENIs will be provisioned.

When running globally scaled serverless applications, note that each [[lambda]] function version or alias creates ENIs in every region where it has been deployed. Also, these ENIs are tied to specific subnets and [[appsync|security]] groups, so manage them carefully to avoid unintended consequences.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service                      | Use Case                                                                   |
| ---------------------------- | ------------------------------------------------------------------------- |
| [[lambda]] [[Srinivas_Notes/VPC|VPC]] Access           | Accessing internal resources ([[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]], [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|EBS]], etc.) securely                |
| AWS App Mesh + [[lambda]]       | Microservices architecture requiring traffic management & observability |
| [[lambda]] without [[Srinivas_Notes/VPC|VPC]] Access  | Public API endpoints, interacting with public AWS services          |

Anti-pattern: Using [[lambda]] [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Access to communicate with public internet-facing services such as [[dynamodb]] or [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. Instead, leverage [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and [[policies]] for fine-grained access control.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be implemented using JSON snippets like below:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
              "ec2:DescribeInstances"
            ],
            "Resource": [
              "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
              "ec2:CreateNetworkInterface",
              "ec2:DeleteNetworkInterface",
              "ec2:DescribeNetworkInterfaces"
            ],
            "Resource": [
              "*"
            ],
            "Condition": {
                "StringEqualsIfExists": {
                    "aws:RequestedVpcServiceOptions": "com.amazonaws.vpce.vpcEndpointServiceId"
                }
            }
        }
    ]
}
```

Cross-account access should be managed through [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] peering, transit gateways, or interface [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|VPC endpoints]] depending on the exact use case. For stricter governance, apply Service Control [[policies]] (SCPs) at the organization level.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling limits depend on the number of concurrent executions per region. To handle high traffic demands, implement exponential backoff strategies in your application code.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute your workloads across multiple regions by deploying duplicate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] and resources in each target region. Ensure proper data replication and consistency mechanisms are in place.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular cost controls include setting up [[Budgets]] and alerts for individual AWS accounts, monitoring usage trends, and applying tags to resources for better visibility. Calculation example:

- Assume a [[lambda]] function configured for [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] access runs 1 million invocations per month with a duration of 500ms and uses 128MB memory configuration.
- Let's also assume the function resides in a single Availability Zone and therefore consumes 1 ENI.
- According to the pricing model, the [[lambda]] cost would be: $0.000000208 * (1,000,000 invocations * 0.5 seconds) * 1 = $0.000104
- Assuming the ENI costs around $0.01/hr, the additional [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] cost would be approximately $0.000709 (assuming 24 hours of operation).

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
Your company is building a new SaaS platform using a microservices architecture. They want to use [[lambda]] but need to ensure proper traffic management and observability. How would you address this requirement?

Correct answer: Leverage AWS App Mesh along with [[lambda]] to provide consistent traffic routing, monitoring, and failure handling capabilities. Incorrect answers might involve using [[lambda]] [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Access or other non-serverless components.

Scenario 2:
A startup wants to build a real-time image processing service using [[lambda]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]]. They expect high demand during certain periods and require a highly available solution. What steps should they take to achieve their goals while keeping costs under control?

Correct answer: Deploy the solution across multiple regions with duplicated [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]] instances. Implement multi-region data replication and consistency mechanisms. Enable [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] access only where necessary to minimize extra costs. Incorrect answers might suggest using a single region or not utilizing [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]].
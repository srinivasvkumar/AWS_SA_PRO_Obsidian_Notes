## Advanced Architecture

The Instance Metadata Service (IMS) is a feature of Amazon [[ec2]] that provides metadata about the running instance. This metadata can be used by applications and services running within an instance to configure themselves or make decisions based on their environment. The IMS operates as a web service accessible through a specific IP address (169.254.169.254) that is available only from the same instance.

At its core, IMS is composed of multiple components responsible for handling metadata requests, managing metadata keys, enforcing [[appsync|security]] [[policies]], and maintaining availability and consistency. These components include the metadata server, metadata database, metadata cache, and metadata request handler.

When designing systems at a global scale, it's essential to understand that IMS has some [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]]. For example, IMS does not support cross-region communication, which means metadata cannot be accessed directly from instances in different regions. However, you can work around this limitation using custom solutions such as storing shared metadata in a centralized database and replicating it across regions.

## Comparison & Anti-Patterns

| Feature                         | IMS                          | Alternatives                              |
| ------------------------------- | ----------------------------- | ------------------------------------------ |
| Accessible within instance       | Yes                           | User Data, Environment Variables          |
| Modifiable during runtime        | Partially (specific keys)    | User Data, Dynamic Configuration Tools     |
| Supports hierarchical structure   | Yes                           | Flat Key-Value Pairs                      |
| Global read access               | No                            | Custom Solutions (centralized databases)    |
| [[appsync|Security]]                        | Complex, requires [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] | Simple (User Data, Environment Variables)   |

Anti-patterns include using IMS for storing sensitive data without proper encryption or applying least privilege principles when granting access to IMS metadata.

## [[appsync|Security]] & Governance

To secure IMS, you must apply complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], restrict cross-account access, and enforce organization-wide Service Control [[policies]] (SCPs). Here's a JSON policy snippet demonstrating how to limit IMDSv2 usage to specific IP addresses:

```json
{
  "Effect": "Deny",
  "Action": ["ec2instancemetadata:Describe"],
  "Resource": "*",
  "Condition": {
    "StringNotEqualsIfExists": {
      "ec2instancemetadata:HttpHeaders": {
        "x-aws-ec2-metadata-token-scope": [
          "https://www.w3.org/2007/xmlschema#IPAddress",
          "10.0.0.0/8"
        ]
      }
    }
  }
}
```

Cross-account access can be managed using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust [[policies]] allowing actions like `ec2:DescribeInstanceMetadata`. To implement organization-wide governance, you may create SCPs that deny unwanted metadata operations or limit the scope of allowed metadata keys.

## Performance & Reliability

IMS imposes throttling limits on metadata requests to ensure fairness and resource allocation among users. To avoid performance issues due to throttling, implement exponential backoff strategies when making repeated requests. In addition, consider [[api-gateway|caching]] frequently accessed metadata within your application to minimize the number of requests.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute instances across Availability Zones (AZs) and Regions while ensuring each instance can access required metadata. If regional redundancy is necessary, consider using custom solutions like centralized databases or message queues to store shared metadata.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]

Granular cost controls for IMS are limited since it is bundled with [[ec2]] compute resources. However, optimize costs by minimizing instance uptime using automation tools like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Spot Instances]], rightsizing instances according to workload requirements, and implementing tagging strategies for better visibility into resource utilization.

## Professional Exam Scenarios

### Scenario A

Suppose you need to provide a highly available solution for sharing metadata between instances in two separate regions. Which architecture would you choose?

#### Solution A-1

Use a centralized database hosted in a third region and set up replication to both target regions. Implement automated failover mechanisms for the database and allow instances to retrieve metadata from the centralized database.

#### Solution A-2

Implement a custom messaging queue system spanning both regions. Allow instances to publish and subscribe to metadata topics in the queue.

#### Justifications

Both approaches have advantages and disadvantages. Solution A-1 relies on existing technology (database replication) but introduces additional complexity. Solution A-2 offers more flexibility and lower latency but requires building and maintaining a custom solution.

### Scenario B

As a principal AWS Solutions Architect, you are tasked with securing the Instance Metadata Service within your organization. What measures would you take to achieve this goal?

#### Solution B-1

Limit IMDSv2 usage to specific IP addresses using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]].

#### Solution B-2

Create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with restricted permissions to access IMS metadata and attach them to relevant instances.

#### Solution B-3

Enforce organization-wide Service Control [[policies]] (SCPs) to prevent unauthorized metadata operations.

#### Justifications

All three solutions help improve [[appsync|security]] by limiting access and enforcing least privilege principles. By combining these measures, you can establish a robust [[appsync|security]] posture for IMS within your organization.
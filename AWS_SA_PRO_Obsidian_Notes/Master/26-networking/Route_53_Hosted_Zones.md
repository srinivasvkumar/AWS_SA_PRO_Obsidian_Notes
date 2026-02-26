#### [[RDS_Instance_Types|1. Advanced Architecture]]

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|Hosted Zones]] are the container for records in a domain or subdomain. They can be simple (single recordsets) or complex (multiple recordsets). A hosted zone is automatically replicated over multiple regions, providing high availability and low latency.

Internally, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] uses a variety of mechanisms for global scale. It employs a globally distributed Domain Name System (DNS) service that offers [[route53|latency-based routing]], directing user requests to the optimal location based on their geographic location. It also supports Geoproximity query routing, allowing you to route traffic based on the location of resources, not the user.

Health checking is another critical component. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] monitor the health of your resources, such as web servers and application endpoints, using active probing. [[route53|Health checks]] can perform HTTP, HTTPS, and TCP checks and be configured to check for specific response codes and text strings.

Under the hood, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] operates at the domain level, not the hosted zone level. This means that when you create a hosted zone, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] does not automatically register the domain name. You must still register your domain through a registrar like Amazon Registrar, GoDaddy, or Namecheap.

#### [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

| Service | Use Cases |
| --- | --- |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] | Global applications, low latency, high availability, [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], health checking. |
| [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] | Content delivery, [[api-gateway|caching]], [[appsync|security]], performance. |
| ALB/NLB | Load balancing, SSL termination, [[elb|session stickiness]]. |

Anti-patterns include using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] for load balancing without understanding the [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]] compared to ALB/NLB. Another mistake is using it for content delivery without considering [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]].

#### [[RDS_Instance_Types|3. Security & Governance]]

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] can be created to manage access to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]]. For example, restricting users from creating, deleting, or updating [[route53|hosted zones]]. Here's an example policy snippet:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:ChangeResourceRecordSets",
                "route53:GetResourceRecordSet"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "StringEqualsIgnoreCase": {
                    "aws:SourceVpc": "vpc-xxxxxxxx"
                }
            }
        }
    ]
}
```

Cross-account access can be granted using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions. Organizational Service Control [[policies]] (SCPs) can enforce restrictions across accounts.

#### [[RDS_Instance_Types|4. Performance & Reliability]]

Throttling limits exist for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] operations. If you exceed these limits, you may receive [[api-gateway|errors]]. To handle this, implement exponential backoff strategies. This involves retrying failed requests with increasing delays between retries.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve configuring multiple record sets with different values. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] will return all values, allowing clients to choose the best option.

#### [[RDS_Instance_Types|5. Cost Optimization]]

Granular cost controls can be implemented by setting up separate [[route53|hosted zones]] for different environments (dev, test, prod) or teams. Remember that each hosted zone has a minimum monthly charge, even if it has no resource record sets.

#### [[RDS_Instance_Types|6. Professional Exam Scenarios]]

Scenario 1: A company has a multi-region application with services hosted in us-east-1 and us-west-2. They want to distribute DNS queries based on geographical location while maintaining high availability. How would you configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] to meet these requirements?

Correct answer: Create a hosted zone and add record sets for each service in both regions. Configure a Geoproximity query routing policy to route users to the nearest region based on network latency. Enable health checking to ensure high availability.

Incorrect answer: Implement a round robin policy. This doesn't take into account the need for geographically aware routing.

Scenario 2: An organization wants to restrict developers from accidentally deleting production [[route53|hosted zones]]. What steps should be taken to achieve this?

Correct answer: Implement an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy denying delete permission for the production hosted zone. Grant this permission only to trusted users or roles.

Incorrect answer: Rely on manual communication and trust. This approach lacks consistency and scalability.
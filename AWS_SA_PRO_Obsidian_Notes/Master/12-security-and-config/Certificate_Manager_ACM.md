**[[RDS_Instance_Types|1. Advanced Architecture]]**

[[acm]]'s internal architecture revolves around certificate issuance, management, and validation. [[acm]] uses Simple Storage Service ([[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]) to store certificates and Amazon's Relational Database Service ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]]) for storing metadata. [[acm]] has built-in integration with Elastic Load Balancing ([[elb]]), [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Application Load Balancer (ALB)]], and [[api-gateway|API Gateway]].

Global Considerations:
- To issue certificates in multiple regions, you need separate certificates for each region. This is because [[acm]] does not replicate issued certificates across regions.
- For cross-region distribution of certificates, consider using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] [[cloudformation|StackSets]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS Backup]] to automate certificate creation in other regions.

Under The Hood Mechanics:
- [[acm]] validates domain ownership through DNS validation, email validation, or manually uploading a CSV file with the required information.
- [[acm]] supports Server Name Indication (SNI) which allows a single load balancer to serve traffic for multiple websites with different domains hosted on it.

**[[RDS_Instance_Types|2. Comparison & Anti-Patterns]]**

| Service | Use Case |
|---|---|
| [[acm]] | Ideal for [[elb]], [[ALB]], and [[api-gateway|API Gateway]]. Supports SNI, automatic renewal, and integration with AWS services. |
| AWS Certificate Manager Private Certificate Authority (ACMPCA) | Create custom private CA hierarchies for internal server [[api-gateway|authentication]] and [[iot]] device verification. |
| [[secrets-manager|AWS Secrets Manager]] / [[parameter-store|AWS Systems Manager Parameter Store]] | Store sensitive data like passwords, tokens, or keys instead of embedding them into applications. |

Anti-Pattern: Using [[acm]] for non-supported resources such as standalone [[ec2]] instances without a supported load balancer. Instead, opt for [[secrets-manager|AWS Secrets Manager]] or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS System Manager]] Parameter Store.

**[[RDS_Instance_Types|3. Security & Governance]]**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "acm:DescribeCertificate",
                "acm:ImportCertificate"
            ],
            "Resource": [
              "*"
            ]
        }
    ]
}
```
Cross-Account Access:
- Enable [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles for cross-account resource access. In the source account, grant necessary permissions via an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy.

Organization Service Control Policy ([[SCP]]):
- Implement centralized control over [[acm]] usage by deploying SCPs at the organization level. Prevent accidental deletion of critical certificates or restrict certificate creation to specific users.

**[[RDS_Instance_Types|4. Performance & Reliability]]**

Throttling Limits:
- [[acm]] provides unlimited free SSL/TLS certificates but enforces throttling limits on operations. Refer to the official documentation for up-to-date quotas.

Exponential Backoff Strategies:
- Implement retry logic when handling exceptions during certificate validation or import. A common strategy involves exponentially increasing wait periods between retries.

HA/DR Patterns:
- Leverage [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and [[ALB]] health monitoring to ensure high availability. For [[dr]], maintain backup certificates and regularly validate their status.

**[[RDS_Instance_Types|5. Cost Optimization]]**

Granular Cost Controls:
- There are no direct costs associated with [[acm]]; however, AWS charges for related services such as [[elb]], [[ALB]], and [[api-gateway|API Gateway]]. Monitor these services for potential cost reduction opportunities.

Calculation Examples:
- Calculate the monthly cost of running an application behind an Application Load Balancer with an [[acm]] certificate:
  `(cost_of_ALB * number_of_ALBs) + (cost_of_API_GW * number_of_APIGWs)`

**[[RDS_Instance_Types|6. Professional Exam Scenarios]]**

Scenario 1:
Your company operates globally and requires a highly available web application served from multiple regions. How would you efficiently manage SSL/TLS certificates while meeting [[appsync|security]] requirements?

Correct Answer:
- Utilize [[acm]] for [[elb]] and [[ALB]] within each region.
- Implement SCPs at the organization level to limit certificate creation to authorized users.
- Set up [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and [[ALB]] health monitoring for high availability.

Incorrect Answer:
- Creating a single [[acm]] certificate for all regions will result in a single point of failure.

Scenario 2:
You want to implement a secure communication channel for your corporate network devices to communicate with your cloud infrastructure. Which AWS service should be used for this purpose?

Correct Answer:
- Use ACMPCA to create a custom private CA hierarchy for internal server [[api-gateway|authentication]] and [[iot]] device verification.

Incorrect Answer:
- Using [[acm]] for internal server [[api-gateway|authentication]] is not recommended due to anti-pattern concerns.
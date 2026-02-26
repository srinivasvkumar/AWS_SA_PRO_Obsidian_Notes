**Advanced Architecture**

At its core, the Well-Architected Tool is a framework that evaluates workloads against five pillars: operational excellence, [[appsync|security]], reliability, performance efficiency, and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]]. The tool offers a set of [[iam|best practices]] and questions to help [[organizations]] evaluate their architecture against these pillars. It operates at an account level, allowing you to evaluate individual accounts or entire [[organizations]].

The Well-Architected Tool uses a question-and-answer format to assess workloads. Each question maps to one or more pillars, with possible remediations suggested if the answer indicates a gap in [[iam|best practices]]. Responses are stored in a centralized repository, which can be used to track progress over time.

Global scale is achieved by using [[organizations|AWS Organizations]] as the foundation for applying Well-Architected principles across multiple AWS accounts. This allows for consistent policy application, cost management, and [[organizations|consolidated billing]].

Internally, the Well-Architected Tool relies on AWS [[AppRunner]] and [[lambda]] under the hood. [[AppRunner]] is used to serve the web application and manage user interactions, while [[lambda]] processes background tasks such as data aggregation and report generation.

**Comparison & Anti-Patterns**

| Service               | When to use                                                              |
| --------------------- | ------------------------------------------------------------------------ |
| Well-Architected Tool | For ongoing assessment and improvement of workloads against [[iam|best practices]] |
| [[trusted-advisor|AWS Trusted Advisor]]   | For real-time monitoring of specific AWS resources                       |
| AWS Personal Health Dashboard | For personalized notifications about service disruptions and events    |

Anti-pattern: Using the Well-Architected Tool only once during initial setup instead of continuously evaluating and improving workloads.

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for the Well-Architected Tool may include permissions to create and update Service Control [[policies]] (SCPs) within an [[AWS Organization]], as well as read-only access to [[CodeCommit]] repositories containing custom questions and remediations.

Example JSON snippet for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "organizations:CreateServiceControlPolicy",
                "organizations:DescribeOrganization",
                "organizations:UpdateServiceControlPolicy"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "codecommit:GetRepository",
                "codecommit:BatchGetRepositories"
            ],
            "Resource": "arn:aws:codecommit:*:123456789012:repository/*"
        }
    ]
}
```
Cross-account access can be established using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles with appropriate trust [[policies]] and permissions. Organization SCPs can enforce restrictions on actions like creating or deleting resources, ensuring compliance with organizational [[policies]].

**Performance & Reliability**

Throttling limits in the Well-Architected Tool are primarily enforced through rate limiting at the [[api-gateway|API Gateway]] layer, which manages incoming requests from users. If throttling occurs, exponential backoff strategies should be employed to ensure reliable operation.

High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns involve deploying the Well-Architected Tool across multiple regions, utilizing services like [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] distributions for load balancing and content delivery.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be implemented by setting up [[billing]] alarms based on usage thresholds and utilizing AWS [[billing|Cost Explorer]] to visualize costs associated with the Well-Architected Tool.

Calculation example:

Assuming a monthly cost of $0.01 per Workload Evaluation and 1,000 evaluations performed, the total cost would be $10 (0.01 * 1000).

**Professional Exam Scenario #1**

Scenario: A large enterprise has adopted the Well-Architected Framework but is struggling with enforcing [[iam|best practices]] across multiple teams and accounts. They want to implement a solution that will automatically apply Service Control [[policies]] (SCPs) based on findings from the Well-Architected Tool.

Correct Answer: Implement an [[organizations|AWS Organizations]] service that includes the necessary SCPs, linked to the appropriate OU(s) within the organization. Configure the Well-Architected Tool to automatically apply SCPs when gaps are identified during workload evaluations.

Incorrect Answer: Create separate [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles for each team with the required [[SCP]] permissions. Provide these roles to the respective teams and instruct them to manually apply SCPs based on the Well-Architected Tool results.

**Professional Exam Scenario #2**

Scenario: An organization wants to limit the number of times the Well-Architected Tool can be invoked daily to prevent potential abuse.

Correct Answer: Set up rate limiting at the [[api-gateway|API Gateway]] level to restrict the number of incoming requests per minute. Implement exponential backoff in case throttling occurs.

Incorrect Answer: Manually limit the number of invocations by configuring a daily quota for each user. Instruct users to monitor their usage and avoid exceeding the quota.
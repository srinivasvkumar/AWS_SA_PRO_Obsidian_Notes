**Advanced Architecture: [[cloudformation]] Drift Detection**

[[cloudformation]] Drift Detection allows you to identify differences between your actual stack resources and their expected counterparts defined in your [[cloudformation]] templates. It operates at the stack or change set level. The detection process is manual, but you can automate it using custom scripts or services like [[config|AWS Config]] Rules.

Internally, Drift Detection works by comparing the Stack Resource Summary (SRS) of your stack, which contains information about all the resources and their properties, against the input template or Change Set. This comparison happens in three phases:

1. Stack Information Collection: [[cloudformation]] retrieves the current state of the stack, including resource types, logical IDs, and physical resource properties.
2. Comparison Phase: The collected data is compared to the provided template or Change Set.
3. Reporting Phase: Discrepancies found during the comparison phase are reported as "drift."

[[RDS_Instance_Types|Global scale considerations]] include managing drift across multiple accounts within an organization. For that purpose, [[organizations|AWS Organizations]] Advanced feature Service Control [[policies]] (SCPs) can be used to enforce centralized control over drift management across member accounts.

**Comparison & Anti-Patterns**

| Issue | Solution |
| --- | --- |
| Using Drift Detection for version controlling infrastructure code | Utilize [[cicd|AWS CodeCommit]], GitHub, or other version control systems instead. |
| Expecting real-time drift identification | Implement periodic scheduled tasks or event-driven solutions such as [[lambda|AWS Lambda]] triggered by [[cloudwatch]] Events. |
| Believing all resources support drift detection | Not all resource types are supported, so consult the official documentation before relying on it. |

**[[appsync|Security]] & Governance**

To ensure secure access to [[cloudformation]] [[cloudformation|stacks]] and perform drift detection, create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy that allows necessary actions (`cloudformation:DetectDrift`, `cloudformation:DescribeStackResources`, `cloudformation:DescribeStackEvents`) while restricting permissions to specific [[cloudformation|stacks]] or regions. Here's a JSON example:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:DetectDrift",
                "cloudformation:DescribeStackResources",
                "cloudformation:DescribeStackEvents"
            ],
            "Resource": [
              "arn:aws:cloudformation:*::stack/my-stack-name/*"
            ]
        }
    ]
}
```

Cross-account access requires additional steps. First, grant the necessary permissions to the source account via an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role or user. Then, assuming the destination account has the required [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] permissions, call the DetectDrift API from the destination account.

Implement Organization SCPs to enforce consistent drift management [[policies]] across member accounts. For instance, deny `cloudformation:DetectDrift` action unless explicitly allowed through another policy.

**Performance & Reliability**

Throttling limits depend on the region and can be found in the official documentation. To handle throttling exceptions, implement exponential backoff strategies in your custom scripts when invoking the DetectDrift API.

For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], distribute your [[cloudformation|stacks]] across different regions and enable drift detection independently in each one. Monitor drift status regularly and maintain separate drift reports for each region.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular cost controls can be achieved by implementing per-account drift detection usage based on your requirements. Calculate costs by considering the number of [[cloudformation|stacks]] and frequency of drift detection checks.

**Professional Exam Scenario #1**

Your company uses [[cloudformation]] to manage its infrastructure across multiple AWS accounts. They want to enforce consistent drift management [[policies]] centrally without limiting individual account administrators' abilities to detect drift. Which approach would meet these requirements?

*Correct Answer*: Implement Organization SCPs to limit drift detection capabilities unless explicitly allowed through another policy. Individual account administrators can still perform drift detection if permitted by the central policy.

Incorrect Answers:
- Implement [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] directly on individual accounts: This solution does not allow centralized management.
- Share a single [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] policy among all accounts: This option does not provide granular control over drift detection permissions.

**Professional Exam Scenario #2**

Your organization manages several thousand [[cloudformation]] [[cloudformation|stacks]] distributed across various AWS accounts. How do you efficiently monitor drift without overwhelming the allowed API calls?

*Correct Answer*: Schedule periodic drift detection tasks or trigger them using events rather than performing real-time drift detection. Additionally, prioritize critical [[cloudformation|stacks]] and stagger the execution of drift detection jobs to avoid exceeding the allowed API calls.

Incorrect Answers:
- Perform real-time drift detection: Real-time drift detection is unnecessary and may lead to excessive API calls.
- Do not schedule periodic drift detection: Regularly monitoring drift is essential to maintaining infrastructure consistency.
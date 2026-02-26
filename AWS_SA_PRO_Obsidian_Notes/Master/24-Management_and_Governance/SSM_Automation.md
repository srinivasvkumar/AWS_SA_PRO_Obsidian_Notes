Advanced Architecture
---------------------

At its core, [[ssm|AWS Systems Manager (SSM)]] Automation is a service that allows you to create executable documents (called "automations") which can be used to perform routine tasks or respond to system events. These automations are written in JSON or YAML and can include AWS CLI commands, AWS SDK calls, scripts, and even conditional logic. Under the hood, [[ssm]] Automation uses [[step-functions|AWS Step Functions]] to manage state transitions and error handling.

When it comes to multi-account strategies, [[ssm]] Automation integrates seamlessly with [[organizations|AWS Organizations]], allowing you to create and manage automations across multiple accounts. This integration enables you to apply automations at a granular level, ensuring that specific actions are taken only in certain accounts or regions. Moreover, [[ssm]] Automation supports Service Control [[policies]] (SCPs) within an organization, enabling centralized control over what actions can be performed using [[ssm]] Automation.

Complex [[api-gateway|integrations]] with other services like [[config|AWS Config]], AWS [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]], and [[lambda|AWS Lambda]] allow [[ssm]] Automation to provide advanced functionality such as automated remediation of configuration drifts, triggering automations based on [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudTrail]] events, and creating custom workflows using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]].

Comparison & Anti-Patterns
---------------------------

| Service | Use Case |
| --- | --- |
| [[ssm]] Automation | Repetitive tasks, responding to system events, and performing ad-hoc tasks |
| [[aws-batch|AWS Batch]] | Running batch jobs that require sustained compute resources |
| [[glue|AWS Glue]] | Data integration and ETL tasks |
| [[lambda|AWS Lambda]] | Event-driven, serverless architecture |

Common anti-patterns include using [[ssm]] Automation for long-running tasks, running interactive sessions, and managing infrastructure deployment. Instead, tools like [[cicd|AWS CodePipeline]], [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]], or AWS Elastic Beanstalk should be used for these scenarios.

[[appsync|Security]] & Governance
----------------------

To ensure [[appsync|security]] and governance, [[ssm]] Automation supports fine-grained [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] that can be applied at various levels, including document, instance, and target level. Here's an example JSON policy snippet that grants permission to execute a specific [[ssm]] Automation document:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ssm:StartAutomationExecution"
            ],
            "Resource": [
                "arn:aws:ssm:*::document/MyAutomationDocument"
            ]
        }
    ]
}
```

Cross-account access can be enabled by adding the appropriate permissions for the source account in the destination account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]. Additionally, [[ssm]] Automation integrates with [[organizations|AWS Organizations]] SCPs, allowing centralized control over what actions can be performed using [[ssm]] Automation.

Performance & Reliability
--------------------------

[[ssm]] Automation has throttling limits that vary depending on the operation. To handle throttling [[api-gateway|errors]], exponential backoff strategies can be employed. For high availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]], [[ssm]] Automation integrates with Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] and [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]], allowing traffic to be routed to the nearest healthy endpoint.

[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
------------------

Granular cost controls can be implemented by using [[ssm]] Automation's resource tagging feature, which enables charging back costs to individual teams or projects. Calculating the cost of an [[ssm]] Automation execution involves understanding the underlying operations being performed, such as API calls or [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]].

Professional Exam Scenarios
---------------------------

**Scenario 1:** A company wants to automatically clean up unused Amazon [[ec2]] instances after they have been terminated by their respective owners. Which approach would satisfy this requirement while minimizing operational overhead?

*Correct Answer:* Use [[ssm]] Automation with AWS [[cloudwatch]] Events to monitor for [[ec2]] instance state changes and trigger the cleanup process when an instance reaches the "terminated" state.

Incorrect Answer: Use [[lambda|AWS Lambda]] with [[cloudwatch]] Events. While this solution will work, using [[ssm]] Automation reduces operational overhead since it provides a more user-friendly interface and eliminates the need for writing code.

**Scenario 2:** A consulting firm manages multiple client accounts and needs to enforce a consistent set of [[appsync|security]] [[policies]] across all accounts. They want to use [[ssm]] Automation to implement these [[policies]]. What steps should they take to achieve this goal?

*Correct Answer:* Create an [[ssm]] Automation document that implements the desired [[appsync|security]] [[policies]] and share it with all relevant client accounts. Then, use [[organizations|AWS Organizations]] SCPs to restrict the actions that can be performed using [[ssm]] Automation.

Incorrect Answer: Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions. While this solution may work, using [[ssm]] Automation and [[organizations]] SCPs ensures consistency and simplifies management compared to manually configuring [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions in each account.
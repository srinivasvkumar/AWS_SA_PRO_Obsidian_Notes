## Advanced Architecture
At its core, Anomaly Detection in AWS is based on statistical analysis and machine learning algorithms. It operates by continuously monitoring AWS cost and usage data to detect unusual trends or outliers that could indicate potential issues or savings opportunities. The service supports both automated and manual methods for identifying anomalies, allowing users to define thresholds and rules based on their specific needs.

[[RDS_Instance_Types|Global scale considerations]] include the ability to aggregate cost and usage data across multiple accounts and regions. This enables [[organizations]] to identify anomalies at a high level as well as drill down into specific resources or services. Under the hood, Anomaly Detection uses a combination of real-time and batch processing techniques to analyze data streams and generate alerts when anomalies are detected.

## Comparison & Anti-Patterns
Here are some scenarios where you might want to use an alternative service instead of Anomaly Detection:

| Service/Tool          | Use Case                                                              |
|-----------------------|--------------------------------------------------------------------------|
| [[billing|AWS Budgets]]           | Fixed budgetary constraints; simple forecasting and alerting             |
| AWS [[billing|Cost Explorer]]     | Visualizing costs over time; ad hoc analysis                            |
| AWS Cost and Usage Report (CUR) | Detailed, granular cost data; custom reporting and analysis         |
| [[cloudwatch|CloudWatch Alarms]]    | Metric-based alerts; near-real-time monitoring                        |

Common anti-patterns when using Anomaly Detection include setting too narrow or broad of a scope, which can lead to either missing important anomalies or receiving excessive false positives. Additionally, relying solely on automatic threshold detection without validating and fine-tuning the settings may result in suboptimal performance.

## [[appsync|Security]] & Governance
To ensure secure access and proper governance of Anomaly Detection, you should follow these [[iam|best practices]]:

- Implement strict [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] to control who can manage and access Anomaly Detection resources. For example, use JSON [[policies]] like this one to restrict access to specific users or groups:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "costexplorer:Describe*",
        "costexplorer:Get*"
      ],
      "Resource": [
        "*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:SourceVpce": "vpce-1234567890abcdef0"
        }
      }
    }
  ]
}
```

- Enable cross-account access by creating an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account and assuming it from the destination account.
- Establish Organization Service Control [[policies]] (SCPs) to enforce centralized control over Anomaly Detection settings and permissions.

## Performance & Reliability
When working with Anomaly Detection, keep throttling limits in mind, such as the maximum number of API requests per second. To mitigate throttling issues, implement exponential backoff strategies using the AWS SDKs. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] (HA/DR) patterns involve distributing Anomaly Detection workloads across multiple regions and ensuring redundancy through backup and restore mechanisms.

## [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls in Anomaly Detection can be achieved by leveraging tags and filtering options to break down costs according to specific criteria. Calculation examples include determining the average daily spend for each tag category or calculating the total monthly cost for a particular department. Here's an example of how to calculate the average daily spend for a specific tag category called `Team`:

```python
import boto3

client = boto3.client('ce', region_name='us-east-1')
response = client.get_cost_and_usage(
    TimePeriod={
        'Start': '2022-01-01T00:00:00Z',
        'End': '2022-01-31T00:00:00Z'
    },
    Granularity='DAILY',
    Metrics=['BlendedCost'],
    GroupBy=[{'Type': 'TAG', 'Key': 'Team'}],
)

total_spend = 0
for day in response['ResultsByTime']:
    for group in day['Groups']:
        total_spend += float(group['Metrics']['BlendedCost']['Amount'])

average_daily_spend = total_spend / len(response['ResultsByTime'])
print("Average Daily Spend: $", round(average_daily_spend, 2))
```

## Professional Exam Scenarios
### Scenario 1
Your organization has multiple AWS accounts managed under a single [[AWS Organization]]. You need to set up an Anomaly Detection solution to monitor costs across all accounts. Which of the following steps would you take?

A) Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] user in each child account with programmatic access to Anomaly Detection, and then grant permission to the parent account's [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role.
B) Set up [[organizations|AWS Organizations]] SCPs to enable cross-account access to Anomaly Detection resources.
C) Enable [[trusted-advisor|AWS Trusted Advisor]] in the master account and configure [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|cost optimization]] recommendations for all member accounts.
D) Create a [[billing]] consolidated account and link all other child accounts to it. Then, create an Anomaly Detection solution within the consolidated account.

Correct Answer: D
Justification: By consolidating bills and creating an Anomaly Detection solution within the consolidated account, you can efficiently monitor costs across all linked child accounts.

Incorrect Answers:
A, B, C: These options do not address the requirement of monitoring costs across all accounts effectively.

### Scenario 2
You have implemented Anomaly Detection for your organization and noticed that certain alerts are frequently triggered due to insufficient thresholds. How would you optimize the configuration to minimize false positives while maintaining sensitivity?

A) Increase the threshold values for affected alerts to reduce the likelihood of triggering them.
B) Implement a custom script that adjusts threshold values based on historical cost data.
C) Utilize AWS Machine Learning services to build a predictive model for more accurate threshold determination.
D) Manually [[nonportable_instructions|review]] and validate each triggered alert before sending notifications.

Correct Answer: B
Justification: Custom scripts can help automate the process of analyzing historical cost data and adjusting threshold values accordingly, reducing false positives without sacrificing sensitivity.

Incorrect Answers:
A, C, D: These options do not sufficiently address the issue of minimizing false positives while maintaining sensitivity.
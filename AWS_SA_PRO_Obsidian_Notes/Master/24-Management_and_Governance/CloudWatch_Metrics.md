**Advanced Architecture**

At its core, [[cloudwatch]] Metrics is a service that collects, processes, and stores metric data from various AWS services, applications, and services provided by third parties. It operates at a global scale, with metrics being published to [[cloudwatch]] namespace regions. However, aggregation of data across namespaces is limited to the region in which it was created. This limitation can be mitigated using custom solutions such as AWS [[kinesis|Kinesis Data Firehose]] or [[lambda|AWS Lambda]] functions.

Metrics are stored in [[cloudwatch]] as time-ordered series of data points called "dimensions." Each dimension consists of one or more pairs of a name and value, making up a unique metric within a specific namespace. The maximum number of dimensions per metric is ten. Under the hood, [[cloudwatch]] Metrics uses an internal system called "Metric Math" to perform mathematical operations on collected metrics.

**Comparison & Anti-Patterns**

| Service                      | Use Case                                                              |
| ---------------------------- | -------------------------------------------------------------------- |
| [[cloudwatch]] Metrics           | Monitoring resource utilization, application performance, and custom metrics. |
| Amazon Managed Service for Prometheus (AMP) | Containerized microservices, Kubernetes workloads, and DevOps tools. |
| AWS Application Performance Management (APM) | End-to-end visibility into production web applications and their underlying components. |

Anti-pattern: Using [[cloudwatch]] Metrics to monitor containerized microservices due to the complexity of creating custom metrics and scaling [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]]. A better choice would be Amazon Managed Service for Prometheus (AMP).

**[[appsync|Security]] & Governance**

Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudwatch:GetMetricData",
                "cloudwatch:GetMetricStatistics",
                "cloudwatch:ListMetrics"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
Cross-Account Access:

1. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with permissions to publish metrics.
2. Attach the necessary permissions policy to the role.
3. In the destination account, create a policy allowing the principal from the source account to put metrics.
4. Add trust relationships between the two accounts.

Organization SCPs:

To restrict specific actions related to [[cloudwatch]] Metrics across multiple accounts, apply an Organization Service Control Policy ([[SCP]]) with the following effect:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DenyCloudWatchMetricsActions",
            "Effect": "Deny",
            "Action": [
                "cloudwatch:DeleteAlarm",
                "cloudwatch:DeleteAnomalyDetector",
                "cloudwatch:DeleteDashboard",
                "cloudwatch:DeleteEventNotification",
                "cloudwatch:DeleteInsightRule",
                "cloudwatch:DeleteMetricStream",
                "cloudwatch:PutMetricData",
                "cloudwatch:PutMetricStream",
                "cloudwatch:UpdateAlarm",
                "cloudwatch:UpdateAnomalyDetector",
                "cloudwatch:UpdateInsightRule"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

**Performance & Reliability**

Throttling Limits:

* PutMetricData API calls: 5 requests per second (automatically adjustable).
* GetMetricData, GetMetricStatistics, ListMetrics API calls: None.

Exponential Backoff Strategies:

* Implement an exponential backoff strategy when handling throttled requests.
* Retry requests with increasing sleep intervals between retries.
* If the request fails after reaching the maximum retry limit, log the error and alert the appropriate team.

HA/DR Patterns:

* Distribute metric streams across multiple regions for higher availability.
* Use [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Route 53]] [[route53|health checks]] and DNS failover for [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] scenarios.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]**

Granular Cost Controls:

* Set individual [[billing]] alarms based on [[cloudwatch]] Metrics for each linked AWS service.
* Enable and configure [[cloudwatch]] Metrics for underutilized resources to identify potential cost savings.

Calculation Examples:

* Calculate the average CPU Utilization metric over the past week:

```python
SEARCH('{AWS/EC2,InstanceId} MetricName="CPUUtilization"', 'Average', 'Auto', {'stat':'Average','start_time':'1w'})
```

**Professional Exam Scenarios**

Scenario 1:

You need to implement a centralized monitoring solution for your organization's AWS infrastructure. The solution should support cross-account access while enforcing granular cost controls. Which of the following steps should you take?

Correct Answer:

1. Create a master [[cloudwatch]] Metrics account.
2. Implement cross-account access between the master account and member accounts.
3. Configure [[cloudwatch]] Metrics in each member account to publish relevant metrics.
4. Set up individual [[billing]] alarms for each linked AWS service in the master account.

Incorrect Answers:

1. Publish all metrics to a single account. This does not allow for granular cost control.
2. Do not set up individual [[billing]] alarms. This reduces visibility and control over costs.

Scenario 2:

Your company runs a large e-commerce platform with strict [[appsync|security]] requirements. You must ensure that no user in any account can delete [[cloudwatch]] Metrics data without proper approval. How can you achieve this?

Correct Answer:

1. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role in the source account with permissions to delete [[cloudwatch]] Metrics data.
2. Attach a permission policy to the role.
3. Apply an [[SCP]] in the Organization to deny deletion actions on [[cloudwatch]] Metrics.

Incorrect Answers:

1. Implement a custom solution to manage metric deletion. This increases complexity and maintenance overhead.
2. Allow unrestricted deletion of [[cloudwatch]] Metrics data. This violates [[appsync|security]] [[iam|best practices]].
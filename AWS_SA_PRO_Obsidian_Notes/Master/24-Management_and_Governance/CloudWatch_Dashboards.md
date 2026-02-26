**Advanced Architecture: [[cloudwatch|CloudWatch Dashboards]] Under the Hood**

[[cloudwatch|CloudWatch Dashboards]] enable customizable views of key metrics and data from multiple services in a single location. They offer a centralized monitoring solution that can be shared across accounts and regions. The following outlines advanced architecture concepts related to [[cloudwatch|CloudWatch Dashboards]]:

* **Data Collection**: [[cloudwatch]] collects data from various AWS services, custom metrics, applications, and logs. Data is stored as time-ordered series of data points called "metrics."
* **[[RDS_Instance_Types|Global Scale Considerations]]**: [[cloudwatch|CloudWatch Dashboards]] support cross-region and cross-account data aggregation. This enables users to build a unified view of their infrastructure, even when resources are distributed globally.
* **Dashboard Components**: A dashboard contains individual components called "widgets" which display specific types of information such as line graphs, stacked graphs, or text. Widgets can be resized, rearranged, and deleted without affecting the underlying metric data.

**Comparison & Anti-Patterns: [[cloudwatch|CloudWatch Dashboards]] vs Alternatives**

| Service                          | Use Case                                                              |
| -------------------------------- | -------------------------------------------------------------------- |
| [[cloudwatch|CloudWatch Dashboards]]            | Centralized monitoring of AWS resources                               |
| Amazon Managed Grafana           | Customizable visualizations, supports non-AWS data sources             |
| Prometheus & Alertmanager       | Scalable monitoring system, ideal for containerized microservices    |
| Elasticsearch, Kibana, Logstash | Full-text search, real-time analytics, and visualizations             |

Anti-patterns include using [[cloudwatch|CloudWatch Dashboards]] as a standalone long-term storage solution or attempting to monitor non-AWS resources without leveraging additional tools like AWS App Mesh or [[api-gateway|API Gateway]] Synthetic Monitoring.

**[[appsync|Security]] & Governance: Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], Cross-Account Access, and Organization SCPs**

To manage access to [[cloudwatch|CloudWatch Dashboards]], you can create [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] with granular permissions. Here's an example JSON policy granting read-only access to [[cloudwatch|CloudWatch Dashboards]]:

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
         "Resource": "*"
      },
      {
         "Effect": "Allow",
         "Action": [
            "cloudwatch:Describe*",
            "cloudwatch:Get*",
            "cloudwatch:List*"
         ],
         "Resource": "arn:aws:cloudwatch:*::dashboard/*"
      }
   ]
}
```

Cross-account access can be enabled by configuring a trust relationship between two AWS accounts. For organization-wide governance, apply Service Control [[policies]] (SCPs) at the organizational unit (OU) level to limit actions like creating or deleting dashboards.

**Performance & Reliability: Throttling Limits, Exponential Backoff Strategies, and HA/DR Patterns**

[[cloudwatch|CloudWatch Dashboards]] have throttling limits including 50 requests per second for listDashboards, getDashboard, and putDashboard operations. To handle these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], implement exponential backoff strategies using SDKs or client libraries.

For high availability (HA) and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] ([[dr]]), leverage [[cloudwatch|CloudWatch Dashboards]]' ability to aggregate data from multiple regions. By storing dashboards in a centrally managed account, you ensure continuity during failures.

**[[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]: Granular Cost Controls and Calculation Examples**

There are no direct costs associated with [[cloudwatch|CloudWatch Dashboards]], but resource usage (e.g., metrics and log ingestion) may incur charges. Implementing granular cost controls involves monitoring and setting alarms based on usage trends.

To illustrate cost calculations, let's consider a scenario where you store 10 million metrics with a monthly charge of $0.001 per 10,000 requests:

* Monthly cost = (10 million / 10,000) \* $0.001 = $1

Keep in mind that actual costs will depend on your specific usage patterns.

**Professional Exam Scenario 1**

Scenario: A large media company wants to monitor its global AWS infrastructure using [[cloudwatch|CloudWatch Dashboards]]. Due to regulatory requirements, they must maintain separate AWS accounts for each country.

Question: Design a solution to allow sharing of [[cloudwatch|CloudWatch Dashboards]] across different AWS accounts while adhering to regulatory restrictions.

Correct Answer: Create a master account with [[cloudwatch|CloudWatch Dashboards]] containing the desired widgets. Set up a role in each regional account allowing it to read data from the master account using [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]. Finally, configure the regional accounts to assume the master account role, enabling them to access the dashboard.

Incorrect Answers:

* Directly share [[cloudwatch|CloudWatch Dashboards]] between accounts without proper roles and [[policies]].
* Store all metrics in one account then provide universal access to all other accounts.

**Professional Exam Scenario 2**

Scenario: Your company operates a mission-critical application requiring high availability and scalability. It uses [[cloudwatch|CloudWatch Dashboards]] for monitoring purposes.

Question: What steps should be taken to ensure the [[cloudwatch|CloudWatch Dashboards]] remain highly available and scalable?

Correct Answer: Distribute resources across multiple regions and aggregate data into a centralized [[cloudwatch]] Dashboard hosted in a dedicated management account. Utilize [[AWS_SA_PRO_Obsidian_Notes/Master/AWS Global Accelerator]] to route traffic through the optimal region based on geographic proximity and latency.

Incorrect Answers:

* Host all resources within a single region, increasing potential downtime due to regional outages.
* Ignore performance optimizations altogether, leading to suboptimal user experiences.
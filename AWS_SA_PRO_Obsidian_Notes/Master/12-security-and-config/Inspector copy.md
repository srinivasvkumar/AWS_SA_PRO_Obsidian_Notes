### Advanced Architecture
At its core, [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/inspector|AWS Inspector]] is an automated [[appsync|security]] assessment service that helps to improve the [[appsync|security]] and compliance of applications deployed on AWS. It does this by automatically assessing the deployed Amazon [[ec2]] instances and applications for vulnerabilities and deviations from [[iam|best practices]]. The [[RDS_Instance_Types|internals]] of [[Inspector]] consist of two main components: a) the IMP ([[Inspector]] MPSecurity Policy) which is an [[ec2]] instance agent responsible for collecting data about the runtime behavior of the application, and b) the [[Inspector]] Service which analyzes the collected data against known [[appsync|security]] rules and guidelines.

The [[Inspector]] service operates at global scale, with multiple geographically dispersed locations around the world. This allows for reduced latency when performing assessments and ensures high availability of the service. Under the hood, [[Inspector]] uses a combination of network traffic analysis and host-based analysis to identify potential [[appsync|security]] issues. The network traffic analysis inspects the traffic between the [[ec2]] instances in a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], while the host-based analysis evaluates the configuration and running processes on the [[ec2]] instances.

### Comparison & Anti-Patterns
Here are some scenarios where [[Inspector]] may not be the best fit and possible alternatives:

| Use Case | [[Inspector]] Alternative | Rationale |
|---|---|---|
| Assessment of static code analysis | AWS [[CodeStar]] | [[Inspector]] focuses on runtime behavior of applications, not static code analysis. |
| Compliance reporting and monitoring | [[config|AWS Config]] | [[Inspector]] provides [[appsync|security]] assessment, whereas [[config|AWS Config]] is focused on resource configurations. |
| Assessment of container workloads | Amazon ECR Image Scanning | [[Inspector]] supports only [[ec2]] instances, not container workloads. |

Common anti-patterns include using [[Inspector]] as a replacement for traditional network or host-based intrusion detection systems, assuming it can detect all vulnerabilities, or relying solely on [[Inspector]] without implementing other [[appsync|security]] measures.

### [[appsync|Security]] & Governance
Complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]] for [[Inspector]] involve granting permissions to various entities like users, groups, and roles. Here's an example JSON policy snippet that grants [[Inspector]] permissions to perform assessments on specified resources:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "inspector:DescribeAssessmentRuns",
                "inspector:DescribeAssessmentTargets",
                "inspector:DescribeFindings",
                "inspector:DescribeRulesPackages",
                "inspector:StartAssessmentRun"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
Cross-account access requires proper permission management through [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and trust relationships. Additionally, [[organizations]] can enforce centralized control over [[Inspector]] settings using Service Control [[policies]] (SCPs) within [[organizations|AWS Organizations]].

### Performance & Reliability
[[Inspector]] has throttling limits that vary based on the region and type of operation. To handle throttling [[api-gateway|errors]], implement exponential backoff strategies that gradually increase the time interval between retries. High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] patterns can be achieved by deploying [[Inspector]] across different regions and ensuring that the underlying [[ec2]] instances are configured for redundancy and failover.

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Cost Optimization]]
Granular cost controls for [[Inspector]] include setting up resource groups and specifying specific [[ec2]] instances to be included in the assessment target. By doing so, you can minimize unnecessary costs associated with assessing unrelated resources. For example, if you have a large number of [[ec2]] instances but only want to assess a subset of them, you can create a resource group containing those instances and then specify that resource group as the assessment target.

### Professional Exam Scenarios
Scenario 1: An organization wants to enable [[Inspector]] for their entire environment, including multiple AWS accounts managed under a single [[AWS Organization]]. What is the recommended approach for achieving this?

Correct answer: Implement [[organizations|AWS Organizations]] and use Service Control [[policies]] (SCPs) to enforce [[Inspector]] usage across member accounts.

Incorrect answer: Create separate [[Inspector]] resources in each account and manually share findings.

Scenario 2: A company experiences throttling issues when using [[Inspector]] due to a large number of concurrent assessment runs. How should they address these [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]]?

Correct answer: Implement exponential backoff strategies and distribute assessment runs across different time intervals.

Incorrect answer: Increase the throttling limits in the [[Inspector]] service.
```yaml
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
```

### Deep-Dive into [[Amazon CloudWatch Synthetics]]

#### UNDER-THE-HOOD MECHANICS:
[[Amazon CloudWatch Synthetics]] uses canaries to monitor application endpoints and APIs, as well as web pages that are part of your application. A canary is a lightweight script that runs on an infrastructureless execution engine in the [[AWS]] cloud. It measures the performance and availability of these resources from different geographical locations.

**Data Flow:**
1. Canaries run at specified intervals.
2. They send requests to the target endpoint (API, web page).
3. The canary collects data such as response times, HTTP status codes, and custom metrics.
4. Data is sent back to [[CloudWatch Synthetics]].
5. Metrics are visualized in [[CloudWatch dashboards]].

**Consistency Models:**
[[CloudWatch Synthetics]] uses eventual consistency for its metrics, which means that the metrics might not be immediately consistent with recent write operations.

#### EXHAUSTIVE FEATURE LIST:
1. **Canary Creation and Management:** Create, update, and delete canaries.
2. **Custom Scripts:** Write custom scripts using [[Node.js]] to interact with your endpoints.
3. **Predefined Steps:** Use predefined steps for common tasks like [[vpc-flow-logs|logging]] in or navigating a web page.
4. **Multiple Geographical Locations:** Run canaries from multiple [[AWS regions]].
5. **Visual Testing:** Capture screenshots and compare them against baseline images to detect UI changes.
6. **Error Handling:** Configure error handling logic within scripts.
7. **Integration with [[cloudwatch]] Metrics, Alarms, and Distributions:**
   - Send metrics to [[CloudWatch Metrics]], use alarms for automated responses to canary findings.
8. **Tagging:** Apply tags to canaries for organizational purposes.
9. **Scheduling:** Set up schedules for running canaries at regular intervals.

#### STEP-BY-STEP IMPLEMENTATION:
1. **Define the Canary Target:**
   - Specify the endpoint URL or web page you want to monitor.
2. **Create a Canary Script:**
   - Use [[Node.js]] for custom scripts, or predefined steps for common tasks.
3. **Configure Canaries:**
   - Choose [[AWS regions]] where canaries will run.
4. **Deploy and Monitor:**
   - Deploy the canary and monitor its performance through [[CloudWatch dashboards]].

#### TECHNICAL LIMITS (2026):
- Maximum number of active canaries per region: 10
- Canary execution time limit: 5 minutes
- Number of steps in a canary script: Up to 100
- Scheduling frequency range: From every 1 minute to once daily

#### CLI & API GROUNDING:
**CLI Commands:**
```sh
aws synthetics create-canaries --name MyCanary \
--code '{"Text": "module.exports.run = function(runner) { ... }"}' \
--execution-role-arn arn:aws:iam::123456789012:role/MyRole \
--schedule {"Expression":"rate(1 minute)"}
```
**API References:**
- `CreateCanaries`: Creates a new canary.
- `UpdateCanaries`: Modifies an existing canary.
- `DeleteCanaries`: Deletes a specified canary.

#### PRICING & TCO:
[[CloudWatch Synthetics]] charges based on the number of virtual CPU-hours used by your canaries. Pricing varies by region, but as of 2026, it is around $1 per vCPU-hour for each active canary. Optimization strategies include:
- Running canaries only during business hours.
- Limiting the number of regions where canaries run.

#### COMPETITOR COMPARISON:
**[[New Relic Synthetics]] vs [[CloudWatch Synthetics]]:**
- **Ease of Use:** New Relic offers a more user-friendly interface for creating and managing tests.
- **Customization:** Both offer Node.js scripts, but New Relic provides more extensive built-in libraries.
- **Pricing Model:** New Relic charges per synthetic monitor, while [[cloudwatch]] Synthetics is based on vCPU-hours.

**[[AppDynamics]] vs [[CloudWatch Synthetics]]:**
- **Features:** AppDynamics includes advanced performance analytics and application tracing.
- **Complexity:** More complex setup compared to [[cloudwatch]] Synthetics.
- **Cost:** Typically more expensive due to additional features, but also provides more granular visibility.

#### INTEGRATION:
[[CloudWatch Synthetics]] integrates seamlessly with [[00_Master_Links_and_Intro|other AWS services]] like [[iam]], [[AWS_SA_PRO_Obsidian_Notes/Master/VPC]], and [[sns]]. You can integrate it with [[sns]] (Simple Notification Service) to send alerts based on canary findings.

#### USE CASES:
1. **Monitoring API Endpoints:** Continuously monitor the health and performance of RESTful APIs.
2. **Web Page Monitoring:** Ensure that web pages are available and respond within expected timeframes.
3. **UI Testing:** Use visual testing features to ensure UI consistency across different sessions or releases.

#### DIAGRAMS:
```plaintext
[Canary Script] -> [CloudWatch Synthetics Execution Engine] -> [Target Endpoint]
                             |
                             v
                        [CloudWatch Metrics]
```

**Trade-offs:**
- **Resource Utilization vs. Cost:** Running canaries from more regions increases cost but improves monitoring comprehensiveness.
- **Complexity vs. Flexibility:** Custom scripts offer flexibility but increase complexity.

#### EXAM TRAPS:
> [!abstract] Exam Tip
Common misconception: Canaries are real-time monitors (eventual consistency applies).
Misunderstanding the difference between synthetic and real user monitoring.
Overlooking the importance of tagging for management purposes.

#### DECISION MATRIX:
| Feature               | Use Case Example                 |
|-----------------------|----------------------------------|
| Custom Scripts        | Monitoring a custom API          |
| Multiple Regions      | Global application coverage      |
| Visual Testing        | Ensuring UI consistency          |

### Audit of Amazon CloudWatch Synthetics

#### Enhancement Requirements:

1. **Distractor Analysis**:
   - **Scenario 1**: If the requirement is to monitor real-time application performance and track end-user experience, other services like [[AWS X-Ray]] or [[Application Load Balancer (ALB) monitoring]] might be more appropriate.
   - **Scenario 2**: When the need is for deep infrastructure-level metrics gathering from servers, [[00_Master_Links_and_Intro|EC2]] instances, or containers, [[CloudWatch Agent]] would be a better fit as it collects detailed system and application performance data.
   - **Scenario 3**: For alerting based on complex business logic that requires triggering actions based on multiple [[cloudformation|conditions]] across different services, [[AWS Lambda]] with [[sns]] might offer more flexibility than Synthetics alone.

2. **The "Most Correct" Logic**:
   - **Performance vs. Cost Trade-offs**:
     - **Performance**: [[cloudwatch]] Synthetics provides high-fidelity monitoring by allowing you to create synthetic checks that mimic user interactions or API calls, ensuring the application's reliability and performance.
     > [!warning] Quota
     - **Cost**: While Synthetics is cost-effective for basic use cases (e.g., checking availability every 1 minute), more frequent checks can increase costs. Monitoring complex applications with many paths might require a larger number of canaries, which could be costly.

3. **Enterprise Governance**:
   > [!check] Implementation
   - **[[organizations|AWS Organizations]]**: Use [[organizations|AWS Organizations]] to manage multiple accounts under one umbrella and apply consistent monitoring [[policies]] across all environments.
   - **Service Control [[policies]] (SCPs)**: Implement SCPs to restrict the creation or modification of [[cloudwatch]] Synthetics canaries to authorized personnel only, ensuring governance and compliance.
   - **[[00_Master_Links_and_Intro|Resource Access Manager (RAM)]]**: Use [[ram]] to share canary data with other AWS accounts within your organization for centralized monitoring and reporting.
   > [!warning] Quota
   - **[[00_Master_Links_and_Intro|CloudTrail]]**: Enable [[00_Master_Links_and_Intro|CloudTrail]] to log all API calls made against [[CloudWatch Synthetics]]. This helps in auditing who created or modified canaries, which is essential for compliance and [[appsync|security]].

4. **Tier-3 Troubleshooting**:
   - **Complex Failure Modes**:
     > [!danger] Distractor
     - **[[kms]] Key Policy Conflicts**: Ensure that the [[kms]] key used to encrypt the logs has appropriate [[policies]] allowing access by [[cloudwatch]] Synthetics. Misconfigured [[kms]] keys can prevent canaries from running or [[vpc-flow-logs|logging]] data.
     - **Network Isolation Issues**: If synthetic checks are failing due to network issues, ensure that [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] endpoint configurations and [[appsync|security]] groups allow outbound traffic for the canary’s script execution.

5. **Decision Matrix: Use Case vs. Solution**:
   | Use Case                          | Recommended Solution                           |
   |-----------------------------------|------------------------------------------------|
   | Monitoring end-user experience    | [[CloudWatch Synthetics]]                     |
   | Infrastructure performance metrics| [[CloudWatch Agent]]                          |
   | Real-time alerting and actions    | [[AWS Lambda]] with [[sns]]                   |
   | API endpoint health               | [[00_Master_Links_and_Intro|Route 53]] [[route53|Health Checks]] or [[ALB]] Target Group     |

6. **Recent Updates**:
   - As of 2026, some new GA features might include improved canary execution capabilities (e.g., more sophisticated scripting support), enhanced integration with [[00_Master_Links_and_Intro|other AWS services]] for automated remediation.
   - Deprecation notices: Keep an eye on any announcements regarding the deprecation of older APIs or functionality within [[CloudWatch Synthetics]]. Regularly updating to newer versions will ensure continued compliance and performance.

### Conclusion:
This [[nonportable_instructions|review]] provides a comprehensive understanding of [[Amazon CloudWatch Synthetics]], its strengths, [[AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/cloudhsm|limitations]], integration with [[00_Master_Links_and_Intro|other AWS services]], and considerations for enterprise governance and troubleshooting complex issues.
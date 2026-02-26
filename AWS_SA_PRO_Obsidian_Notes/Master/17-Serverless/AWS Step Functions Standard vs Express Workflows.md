```markdown
---
tags: [AWS/SAP-C02, DataArchitecture]
status: to_review
---

### AWS Step Functions Standard vs Express Workflows

#### Under-the-Hood Mechanics:
**[[Standard Workflow]]:**
- **Data Flow:** Data is stored and processed in memory and persisted to durable storage.
- **Consistency Model:** Eventual consistency for large state transitions, but strong consistency within a single execution.
- **Underlying Infrastructure:** Utilizes [[AWS Lambda]], [[Amazon ECS tasks]], or other AWS services as steps.

**[[Express Workflow]]:**
- **Data Flow:** Data is primarily held in memory with snapshots taken periodically. Optimized for low latency.
- **Consistency Model:** Similar to Standard but optimized for high throughput and low latency.
- **Underlying Infrastructure:** Uses a lightweight execution engine optimized for short-lived tasks (up to 5 minutes).

#### Exhaustive Feature List:
**Standard Workflow:**
1. Supports long-running executions up to one year.
2. Can handle complex state transitions and error handling.
3. Integrates with [[AWS CloudWatch]], [[AWS X-Ray]], AWS Lambda, and other services.
4. Offers detailed logging and traceability through [[CloudWatch]].

**Express Workflow:**
1. Optimized for short-lived tasks (up to 5 minutes).
2. Supports parallel processing and concurrency up to thousands of executions per second.
3. Provides low-latency responses, ideal for real-time applications.
4. Limited error handling compared to Standard Workflows due to the lightweight nature.

#### Step-by-Step Implementation:
**Standard Workflow:**
1. Design state machine using [[AWS Management Console]] or AWS CLI/API.
2. Define states and transitions within the state machine.
3. Integrate with other AWS services via service integrations.
4. Deploy and monitor executions through [[CloudWatch]].

**Express Workflow:**
1. Create a lightweight state machine optimized for short tasks.
2. Define steps that are typically less than 5 minutes in duration.
3. Use parallel processing features to handle high throughput requirements.
4. Monitor performance using [[CloudWatch Metrics]] for low latency insights.

#### Technical Limits (2026):
**Standard Workflow:**
- Execution duration up to one year.
- Maximum execution history size of 8 MB per execution.
- Up to 50,000 in-flight executions concurrently across all standard workflows in a region.

**Express Workflow:**
- Execution duration limited to 5 minutes.
- Limited state machine complexity and error handling capabilities.
- High concurrency support (up to thousands of executions per second).

#### CLI & API Grounding:
**Standard Workflow:**
```bash
aws stepfunctions create-state-machine --name MyStateMachine --definition file://statemachine.json --role-arn arn:aws:[[00_Master_Links_and_Intro|iam]]::123456789012:role/MyRole
```

**Express Workflow:**
```bash
aws stepfunctions create-state-machine --name MyExpressWorkflow --definition file://expressworkflow.json --type EXPRESS --role-arn arn:aws:[[00_Master_Links_and_Intro|iam]]::123456789012:role/MyRole
```

#### Pricing & TCO:
**Standard Workflow:**
- Charges per execution, based on state transitions and duration.
- Cost-effective for long-running workflows.

**Express Workflow:**
- Designed to be cheaper than Standard Workflows for short tasks.
- Optimized pricing model for high throughput applications.

#### Competitor Comparison:
**AWS Step Functions vs Azure Logic Apps:**
- **Step Functions:** Provides a more robust state machine approach with detailed error handling and complex transitions.
- **Logic Apps:** Offers a workflow orchestration service with a more visual interface but less control over state management compared to [[Step Functions]].

#### Integration:
**Standard Workflow:**
- Integrates seamlessly with AWS CloudWatch, IAM, VPC, Lambda, ECS, etc.
- Supports detailed monitoring and logging through [[CloudWatch Metrics]] and Logs.

**Express Workflow:**
- Similar integrations but optimized for low-latency applications.
- Uses [[CloudWatch Metrics]] to monitor performance and throughput.

#### Use Cases:
**Standard Workflow:**
1. Complex order processing systems with multiple steps.
2. Long-running data transformation pipelines.

**Express Workflow:**
1. Real-time analytics dashboards.
2. Short-lived tasks in IoT device management.

> [!abstract] Exam Tip
- Confusing Standard vs. Express Workflows' execution duration limits.
- Misunderstanding the role of CloudWatch in monitoring both workflows.
- Overlooking the pricing differences between long-running and short-lived tasks.

#### Decision Matrix: Features vs. Use Cases
| Feature                  | Standard Workflow                           | Express Workflow                          |
|--------------------------|---------------------------------------------|-------------------------------------------|
| Execution Duration       | Up to one year                              | Up to 5 minutes                           |
| Concurrency              | Limited to 50K in-flight executions         | High concurrency support (thousands/second)|
| Error Handling           | Detailed                                    | Basic                                     |
| Integration              | [[AWS CloudWatch]], IAM, VPC                | Similar but optimized                     |

> [!check] Implementation
- **Design a workflow for long-running data processing:** Identify appropriate state transitions, error handling mechanisms.
- **Optimize short-lived tasks in real-time applications:** Highlight Express Workflow benefits and integration with [[CloudWatch Metrics]].

#### 2026 Updates:
- Ensure all quotas and pricing are aligned with the latest AWS updates.
- Incorporate any new features or limitations introduced in 2026.

> [!warning] Quota
- Both Standard and Express Workflows have limits on the number of concurrent executions. Exceeding these can lead to throttling, affecting performance.

#### Exam Scenarios:
1. **Design a workflow for long-running data processing:** Identify appropriate state transitions, error handling mechanisms.
2. **Optimize short-lived tasks in real-time applications:** Highlight Express Workflow benefits and integration with [[CloudWatch Metrics]].

> [!danger] Distractor
- When dealing with long-running tasks (e.g., >5 days) where the workflow does not require frequent updates and you want to minimize cost, Step Functions Express Workflows are limited to a maximum execution duration of 5 minutes per state transition. For longer-running workflows, Standard Workflows should be used.
```
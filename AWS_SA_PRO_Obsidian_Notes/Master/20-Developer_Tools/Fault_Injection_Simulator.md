## Advanced Architecture

At its core, Fault Injection Simulator (FIS) is an AWS managed service that allows developers to test their system's resilience by introducing various types of failures. The primary components of FIS include:

- **Fault Targets**: These represent the resources or services in your application stack that you want to target during fault injection. They can be [[ec2]] instances, Amazon [[ecs]] tasks, or even entire AWS Regions.
- **Fault Actions**: These define the specific type of failure to introduce, such as network latency, CPU usage spikes, or complete resource termination.
- **Experiments**: An Experiment is a collection of one or more fault actions aimed at a set of targets.

FIS operates globally, allowing you to simulate faults across multiple regions concurrently. This capability ensures that your systems are tested under real-world [[cloudformation|conditions]], particularly when dealing with geographically dispersed applications.

Internally, FIS uses several mechanisms to ensure high availability and reliability:

- **Multi-Region Deployment**: FIS deploys its control plane components across multiple AWS Regions, providing redundancy and ensuring uninterrupted service availability.
- **Automatic Failover**: If any component within FIS experiences an outage, traffic is automatically redirected to healthy endpoints without manual intervention.
- **Data Isolation**: Each account has its own dedicated data store, isolating customer data and preventing potential cross-contamination issues.

## Comparison & Anti-Patterns

While FIS provides robust fault injection capabilities, it may not always be the best solution. Here are some comparisons and anti-patterns to consider:

| Service           | Advantages                                                                   | Disadvantages                                                      | Use Cases                                             |
|-------------------|-------------------------------------------------------------------------------|----------------------------------------------------------------------|-------------------------------------------------------|
| Fault Injection Simulator (FIS)    | Allows fine-grained fault injection, supports multi-region testing          | Limited support for non-EC2 resources (e.g., [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|RDS]], [[lambda]]), requires [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] permissions | Testing resiliency of distributed applications |
| Chaos Engineering Tools (Gremlin, Chaos Toolkit) | Supports wider range of resource types, customizable chaos experiments | Less integrated with [[Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]], steeper learning curve | More flexible fault injection requirements |
| Load Testing Tools (Truffle, Artillery) | Primarily focused on performance testing, not fault tolerance | Not designed for fault injection, limited fault simulation capabilities | Performance benchmarking, stress testing |

Common design mistakes when using FIS include:

- Insufficient scope: Only targeting a subset of resources instead of considering the entire application stack.
- Overlooking dependencies: Neglecting to account for downstream impacts on other services and resources.
- Poor documentation: Failing to document FIS experiments, making it difficult to track which tests were executed and their outcomes.

## [[appsync|Security]] & Governance

To secure access to FIS, you should implement complex [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]], cross-account access, and organization service control [[policies]] (SCPs):

### [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]

Here's an example JSON policy granting permission to create and modify experiments for a specific user:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "fis:CreateExperiment",
        "fis:DeleteExperiment",
        "fis:DescribeExperiment",
        "fis:GetFaultStatus",
        "fis:UpdateExperiment"
      ],
      "Resource": "arn:aws:fis:*::user/*/experiment/*"
    }
  ]
}
```

### Cross-Account Access

To allow users from another account to perform fault injection tests, specify the source account ID in the resource ARN:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "123456789012"
      },
      "Action": [
        "fis:CreateExperiment",
        "fis:DeleteExperiment",
        "fis:DescribeExperiment",
        "fis:GetFaultStatus",
        "fis:UpdateExperiment"
      ],
      "Resource": "arn:aws:fis:*::user/111122223333/experiment/*",
      "Condition": {
        "StringEquals": {
          "aws:SourceVp
```
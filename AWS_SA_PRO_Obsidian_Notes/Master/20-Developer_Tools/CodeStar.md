## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, AWS CodeStar enables collaboration between different members of a software development team by providing a unified user interface to manage code, build, test, and deploy applications. CodeStar supports various programming languages, including Python, Java, Ruby, and JavaScript, and integrates with popular development tools such as [[cicd|AWS CodeCommit]], [[cicd|AWS CodeBuild]], [[cicd|AWS CodeDeploy]], and [[cicd|AWS CodePipeline]].

CodeStar uses a project template model that includes sample code, a build specification file, and a pipeline definition file. The project templates can be customized based on specific requirements. CodeStar creates an [[cicd|AWS CodePipeline]] when a new project is created, which automates the build, test, and deployment process.

Internally, CodeStar uses AWS Identity and Access Management ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]]) roles and permissions to enable users to perform actions on their behalf. CodeStar also integrates with [[organizations|AWS Organizations]] to allow creating and managing projects across multiple AWS accounts.

Global scale is achieved through AWS's highly available and redundant infrastructure. CodeStar leverages Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for storing project files, Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] for content delivery, and [[lambda|AWS Lambda]] for running build and test scripts. CodeStar also integrates with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|AWS CloudFormation]] for infrastructure provisioning and [[AWS Systems Manager]] for managing resources.

## [[RDS_Instance_Types|2. Comparison & Anti-Patterns]]

The following table compares CodeStar with other developer tools in the AWS ecosystem:

| Service | Primary Use Case |
| --- | --- |
| CodeStar | Collaboration and workflow management for software development teams. |
| [[CodeCommit]] | Source control management. |
| CodeBuild | Build automation. |
| CodeDeploy | Application deployment. |
| CodePipeline | Continuous integration and continuous delivery ([[cicd|CI/CD]]). |
| [[CodeArtifact]] | [[Artifact]] management. |

An anti-pattern for using CodeStar is when a single account is used for all projects. This approach may lead to resource contention and difficulty in managing access and permissions. A better approach would be to create separate AWS accounts for each project or group of related projects.

## [[RDS_Instance_Types|3. Security & Governance]]

CodeStar provides several [[appsync|security]] features, including [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions, AWS Key Management Service ([[kms]]) encryption, and [[organizations|AWS Organizations]] integration. Here's an example JSON policy that allows a user to view and edit a CodeStar project:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "codestar:DescribeProject",
                "codestar:DescribeProjectMembers",
                "codestar:ListProjects"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "codestar:AssociateTeamMember",
                "codestar:DisassociateTeamMember"
            ],
            "Resource": [
              "arn:aws:codestar:*:<account_id>:project/*/team/*"
            ]
        }
    ]
}
```
Cross-account access can be enabled by configuring [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] roles and permissions in both the source and destination accounts. CodeStar also integrates with [[organizations|AWS Organizations]] to provide centralized management of member accounts and Service Control [[policies]] (SCPs).

## [[RDS_Instance_Types|4. Performance & Reliability]]

CodeStar has built-in throttling limits for API calls and concurrent operations. To avoid hitting these limits, developers should implement exponential backoff strategies in their code. Here's an example implementation in Python:
```python
import time

def call_api(api_client):
    try:
        api_client.do_something()
    except api_client.exceptions.ThrottlingException:
        wait_time = calculate_wait_time(1)
        print("Throttled. Waiting for {} seconds.".format(wait_time))
        time.sleep(wait_time)
        call_api(api_client)

def calculate_wait_time(max_wait_time):
    return min(max_wait_time * 2, max_wait_time + 5)
```
High availability and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|disaster recovery]] (HA/DR) patterns can be implemented by using AWS CodeStar in combination with other services such as [[cicd|AWS CodeCommit]], [[cicd|AWS CodeBuild]], [[cicd|AWS CodeDeploy]], and [[cicd|AWS CodePipeline]]. These services provide automatic failover and redundancy capabilities to ensure maximum uptime and availability.

## [[RDS_Instance_Types|5. Cost Optimization]]

CodeStar provides granular cost controls through its integration with [[organizations|AWS Organizations]]. By creating separate AWS accounts for each project or group of related projects, developers can easily track and manage costs associated with each project. Additionally, CodeStar integrates with AWS [[billing|Cost Explorer]] to provide detailed cost breakdowns and usage metrics.

Here's an example calculation for the cost of using CodeStar:

Assume a CodeStar project that includes the following resources:

* [[cicd|AWS CodeCommit]]: $1 per month
* [[cicd|AWS CodeBuild]]: 10 builds per day, each taking 10 minutes to complete. Assuming a build cost of $0.00001 per minute, the monthly cost would be $1.80.
* [[cicd|AWS CodeDeploy]]: 5 deployments per month, assuming a cost of $0.0001 per minute, the monthly cost would be $0.05.
* [[cicd|AWS CodePipeline]]: Assume a cost of $0.0001 per pipeline execution, and assume 10 executions per month. The monthly cost would be $0.01.

Adding up these costs, the total monthly cost for the CodeStar project would be approximately $2.86.

## [[RDS_Instance_Types|6. Professional Exam Scenarios]]

### Scenario 1

Suppose you need to implement a [[cicd|CI/CD]] pipeline for a new application using CodeStar. The application consists of a frontend written in React, a backend written in Node.js, and a database using MongoDB. How would you set up the pipeline?

Correct answer:

1. Create a new CodeStar project and select the appropriate project template based on the technologies used in the application.
2. Configure the pipeline settings, including the source code repository ([[cicd|AWS CodeCommit]]), build provider ([[cicd|AWS CodeBuild]]), and deployment provider ([[cicd|AWS CodeDeploy]]).
3. Define the build steps for the frontend and backend components, including transpiling the React code and installing Node.js dependencies.
4. Configure the deployment settings for the backend component, including the serverless framework ([[lambda|AWS Lambda]]) and the [[api-gateway|API Gateway]] endpoint.
5. Set up the pipeline to automatically trigger on code changes, run tests, and deploy the updated application.

Incorrect answer:

Using [[cicd|AWS CodeBuild]] instead of CodeStar for building the application. While CodeBuild can be used for building and testing applications, it lacks the collaboration and workflow management features provided by CodeStar.

### Scenario 2

Suppose you need to grant a third-party vendor access to your CodeStar project to fix a critical bug. What is the recommended way to grant them access while ensuring they don't have unnecessary privileges?

Correct answer:

1. Create an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role with the necessary permissions to access the CodeStar project.
2. Add the third-party vendor as a team member in CodeStar and assign them the [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role.
3. Remove the third-party vendor from the CodeStar project once the bug is fixed.

Incorrect answer:

Sharing the root account credentials with the third-party vendor. This approach is not recommended because it violates the principle of least privilege and increases the risk of unauthorized access.
---
aliases:
- "Best practices"
- "IAM Roles vs Resource Based Policies"
- "IAM: Identity and Access Management"
---

# IAM: Identity and Access Management

- When accessing AWS, the root account should **never** be used. Users must be created with the proper permissions. [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] is central to AWS
- **Users**: A physical person
- **Groups**: Functions (admin, devops) Teams (engineering, design) which contain a group of users
- **Roles**: Internal usage within AWS resources
    - **Cross Account Roles**: roles used to assumed by another AWS account in order to have access to some resources in our account
- **[[policies]] (JSON documents)**: Defines what each of the above can and cannot do. **Note**: [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] has predefined managed [[policies]]
    - There are 3 types of [[policies]]:
        - AWS Managed
        - Customer Managed
        - Inline [[policies]]
- **Resource Based [[policies]]**: [[policies]] attached to AWS services such as [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]], [[sqs]]

## IAM Roles vs Resource Based Policies

- When we assume a role (user, application or service), we give up our original permission and take the permission assigned to the role
- When using a resource based policy, principal does not have to give up any permissions
- Example: user in account A needs to scan a [[dynamodb]] table in account A and dump it in an [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] bucket in account B. In this case if we assume a role in account B, we wont be able to scan the table in account A
  
## Best practices

- One [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] User per person **ONLY**
- One [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] Role per Application
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] credentials should **NEVER** be shared
- Never write [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] credentials in your code. **EVER**
- Never use the ROOT account except for initial setup
- It's best to give users the minimal amount of permissions to perform their job
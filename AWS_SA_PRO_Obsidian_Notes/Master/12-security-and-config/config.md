---
aliases:
- "AWS Config"
- "AWS Config Remediations"
- "Conformance Packs"
---

# AWS Config

- AWS Config has 2 main jobs:
    - Primary: record configuration changes over time on AWS resources. Every time a configuration is changed on a resource a configuration item is created which stores the change at that specific point in time
    - Secondary: auditing of changes, compliance with standards
- Config does not prevent changes from happening! It is not a permissions product or a protections product. Even if we define standards for resources, Config can check the compliance against those standards, but it does not prevent us from braking those standards
- Config is a regional service, supports cross-region and cross-account aggregations
- Changes can generate [[sns]] notifications and near-realtime events via [[eventbridge]] and [[lambda]]
- Config stores changes historically in a consistent format in an [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] product bucket
- Config recording has to be manually enabled!
- Config Rules:
    - Can be AWS managed ones or user defined using [[lambda]]
    - Resources are evaluated against these rules determining if there are compliant or non-compliant
    - Custom Rules use [[lambda]], the function does the evaluation and returns the information back to Config
- Config can be integration to [[eventbridge]] which can be used to invoke [[lambda]] functions for automatic remediation
- Config can also have integration with [[ssm]] to remediate issues
- AWS Config architecture:
    ![AWS Config architecture](AWS_SA_PRO_Obsidian_Notes/Master/12-security-and-config/images/AWSConfig.png)

## AWS Config Remediations

- Nom-compliant resources can be remediated automatically using System Manager Automation documents
- AWS Config provides a set of managed automation documents with remediation actions. We can create custom remediation documents

## Conformance Packs

- A conformance pack is a collection of AWS Config rules and remediation actions that can be deployed as a single entity in an account/region or across an [[AWS Organization]]
- Conformance packs are created by authoring a YAML template that contains a list AWS Config rules and remediation actions
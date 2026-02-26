---
aliases:
- "Amazon Kendra"
- "Key Concepts"
- "Others"
---

# Amazon Kendra

- Is an intelligent search service
- It's primary aim is to be designed to mimic interacting with a human expert
- Supports wide range if question types such as:
    - Factoid: Who, What, Where
    - Descriptive: How do I get my cat to spot being a jek?
    - Keyword: What time is the keynote address (address can have multiple meaning) - [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Kendra|Kendra]] helps determine intent

## [[kms|Key Concepts]]

- **Index**: searchable data organized in an efficient way
- **Data Source**: where the data lives, [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Kendra|Kendra]] connects and indexes from this location. Example of locations are: [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]], Confluence, Google [[workspaces]], [[rds]], OneDrive, Salesforce, [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Kendra|Kendra]] Web Crawler, Workdocs, [[fsx]], etc.
- We configure [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Kendra|Kendra]] to synchronize a data source with an index based on a **schedule**. This should keep the index current
- **Documents**: can be structured (FAQs) and unstructured (HTML, PDF, etc.)

## Others

- [[AWS_SA_PRO_Obsidian_Notes/Master/Machine_Learning/Kendra|Kendra]] integrates with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]]: [[iam]], Identity Center (SSO) and [[AWS_SA_PRO_Obsidian_Notes/Master/05-compute/others|others]]
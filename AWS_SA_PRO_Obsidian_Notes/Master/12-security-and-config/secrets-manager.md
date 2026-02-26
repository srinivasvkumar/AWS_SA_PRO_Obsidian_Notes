---
aliases:
- "AWS Secrets Manager"
---

# AWS Secrets Manager

- It does share functionality with [[ssm]] Parameter Store
- [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] is designed specifically for secrets, example passwords, API Keys
- It is usable via Console, CLI, API or SDK's
- It supports the automatic rotation of secrets using a [[lambda]] functions
- For certain AWS services, [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Secrets Manager]] offers direct integration, such as [[rds]] (automatic synchronization when the secrets are rotated)
- Secrets are encrypted using [[kms]]
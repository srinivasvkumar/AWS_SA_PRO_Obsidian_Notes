---
aliases:
- "Amazon Macie"
- "Macie Findings"
- "Macie Identifiers"
---

# Amazon Macie

- It is a [[rds|data security]] and data privacy service
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Macie|Macie]] is a service to discover, monitor and protect data stored in [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] buckets
- Once enabled and pointed to buckets, [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Macie|Macie]] will automatically discover data and categorize it as PII, PHI, Finance etc.
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Macie|Macie]] is using data identifier. There are 2 [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|types of data]] identifier:
    - Managed Data Identifier: built-in, can use machine learning, pattern matching to analyze and discover data. It is designed to detect sensitive data from many countries
    - Custom Data Identifier: created by clients, they are proprietary to accounts and they are regex based
- Discovery Jobs: these jobs will use data identifiers to manage and search for sensitive content. They will generate findings which can be used for integration with [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]] (ex: [[appsync|Security]] Hub from where findings can be passed to Event Bridge) in order to do automatic remediation
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Macie|Macie]] uses multi account architecture: one account is the administrator account which can used to manage [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Macie|Macie]] within the member accounts to discover sensitive data
- This multi-account structure can be done with AWS [[organizations]] or by explicitly inviting accounts in [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Macie|Macie]]

## Macie Identifiers

- Data Discovery Jobs: analyzes data in order to determine wether the objects contain sensitive data. This is done using data identifiers
- **Managed Data Identifiers**:
    - Created and managed by AWS
    - Can be used to detect a growing list of common sensitive data types: credentials, financial data, health data, personal identifiers (addresses, passports, etc.)
- **Custom Data Identifiers**:
    - Can be created by us, AWS account users/owners
    - They are using regex patterns to match data
    - We can add optional keywords: optional sequences that need to be in the proximity to regex match
    - Maximum Match Distance: how close keywords are to regex pattern
    - We can also include ignore words

## Macie Findings

- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Macie|Macie]] will produce 2 types of findings:
    - **Policy Findings**: are generated when the [[policies]] or settings are changed in a way that reduces the [[appsync|security]] of the bucket after [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Macie|Macie]] is enabled
    - **Sensitive Data Findings**: generated when sensitive data is identified based on identifiers
- Types if policy findings:
    - `Policy:IAMUser/S3BlockPublicAccessDisabled`: all bucket-level block public access settings were disabled for the bucket
    - `Policy:IAMUser/S3BucketEncryptionDisabled`: default encryption settings for the bucket were reset to default Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] encryption behavior, which is to encrypt new objects automatically with an Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] managed key
    - `Policy:IAMUser/S3BucketPublic`: an ACL or bucket policy for the bucket was changed to allow access by anonymous users or all authenticated AWS Identity and Access Management ([[iam]]) identities
    - `Policy:IAMUser/S3BucketSharedExternally`: an ACL or bucket policy for the bucket was changed to allow the bucket to be shared with an AWS account that's external to (not part of) your organization
- Types of sensitive data findings:
    - `SensitiveData:S3Object/Credentials`: object contains sensitive credentials data, such as AWS secret access keys or private keys
    - `SensitiveData:S3Object/CustomIdentifier`: object contains text that matches the detection criteria of one or more custom data identifiers
    - `SensitiveData:S3Object/Financial`: object contains sensitive financial information, such as bank account numbers or credit card numbers
    - `SensitiveData:S3Object/Multiple`: object contains more than one category of sensitive data
    - `SensitiveData:S3Object/Personal`: object contains sensitive personal information—personally identifiable information (PII) such as passport numbers or driver's license identification numbers, personal health information (PHI) such as health insurance or medical identification numbers, or a combination of PII and PHI
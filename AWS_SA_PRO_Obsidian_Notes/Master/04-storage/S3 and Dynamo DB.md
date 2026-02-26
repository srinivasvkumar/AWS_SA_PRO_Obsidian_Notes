
---
tags: [storage, database, networking, service:s3, service:dynamodb, domain:storage, domain:databases, concept:gateway-endpoint, concept:public-endpoint, concept:iam-policy]
---

### 1. The Services are Public (Public Endpoints)

By default, both [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and [[dynamodb]] are region-wide, public AWS services. This means they live **outside** of your Virtual Private Cloud ([[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]]), and their API endpoints (e.g., `s3.us-east-1.amazonaws.com` or `dynamodb.us-east-1.amazonaws.com`) are resolvable over the public internet.

You do not need a [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], a specific network, or a [[AWS_SA_PRO_Obsidian_Notes/Master/VPN|VPN]] to reach them. Anyone with internet access can ping their endpoints.

### 2. The Data is Completely Private (Secure by Default)

Even though the front door faces the public internet, the building is locked tight.

- **[[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]:** By default, all newly created [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] buckets have "Block Public Access" enabled. No one can read, write, or list the objects inside the bucket unless they have explicit AWS Identity and Access Management ([[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]]) permissions or a highly specific bucket policy granting them access.
    
- **[[dynamodb]]:** There is no concept of a "public" [[dynamodb]] table. Access to [[dynamodb|DynamoDB tables]] is strictly controlled by [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] [[policies]]. You must have cryptographically signed requests (using valid AWS credentials) to interact with the data.
    

---

### How Your Private [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Accesses These Services

Because [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] and [[dynamodb]] are public services that live outside your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]], resources inside a private subnet (like an [[ec2]] instance without a public IP) cannot reach them directly by default.

To bridge this gap securely without sending traffic over the public internet, AWS uses **[[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] [[AWS_SA_PRO_Obsidian_Notes/Master/03-networking/privatelink|Gateway Endpoints]]**.

A [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] Gateway Endpoint acts as a secure, private tunnel from your [[AWS_SA_PRO_Obsidian_Notes/Master/VPC|VPC]] directly to [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] or [[dynamodb]], keeping all your traffic entirely within the AWS global network.
---
aliases:
- "IAM Identity Center (Successor to AWS Single Sing-On - SSO)"
- "IAM Identity Center Architecture"
---

# IAM Identity Center (Successor to AWS Single Sing-On - SSO)

- A way to use existing enterprise identity store with AWS
- Allows to centrally manage SSO access to multiple AWS accounts and external business applications as well
- Replaces the historical uses cases provided by [[AWS_SA_PRO_Obsidian_Notes/Master/02-identity/saml|saml]] 2.0
- Flexible Identity store: where identities are stored. Allows for external identities to be swapped with AWS credentials
- [[iam]] Identity Center supports the following type of identity stores:
    - Built-in identity store
    - AWS Managed Microsoft AD
    - On-premise Microsoft AD (Two way trust or AD Connector)
    - External Identity Provider - [[AWS_SA_PRO_Obsidian_Notes/Master/02-identity/saml|saml]] 2.0
- [[iam]] Identity Center is preferred by AWS for any "workforce" (enterprise) identity federation over the traditional [[AWS_SA_PRO_Obsidian_Notes/Master/02-identity/saml|saml]] 2.0 based identity federation

## IAM Identity Center Architecture

![IAM Identity Center](AWS_SA_PRO_Obsidian_Notes/Master/02-identity/images/AWSSSO.png)

- A requirement of [[iam]] Identity Center is that we should have a valid [[AWS Organization]] created
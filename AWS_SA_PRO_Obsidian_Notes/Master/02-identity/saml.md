---
aliases:
- "SAML 2.0 Federation"
- "SAML 2.0 Identity Federation"
- "SAML 2.0 Identity Federation Authentication Process - API Access"
- "SAML 2.0 Identity Federation Authentication Process - AWS Console Access"
---

# SAML 2.0 Identity Federation

- Federation lets user outside of AWS to assume a temporary role for access AWS resources
- These users assume identity provider access role
- [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]] 2.0 - [[appsync|Security]] Assertion Markup Language
- [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]] 2.0 allows to **indirectly** use on-premise identities with AWS (console and CLI)
- [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]] 2.0 based identity federation is used when we have and enterprise based identity provider which is [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]] 2.0 compatible
- [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]] 2.0 based federation is ideal when we have an existing identity management team managing access to other services including AWS
- If we are looking to maintain a single source of truth and/or we have more than 5000 users, [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]] 2.0 based federation is recommended to be used
- Federation is using [[iam]] Roles and AWS Temporary Credentials (12 hours validity)

## SAML 2.0 Identity Federation Authentication Process - API Access

![SAML 2.0 Federation API](AWS_SA_PRO_Obsidian_Notes/Master/02-identity/images/SAML2.0FederationAPI.png)

## SAML 2.0 Identity Federation Authentication Process - AWS Console Access

![SAML 2.0 Federation Console](AWS_SA_PRO_Obsidian_Notes/Master/02-identity/images/SAML2.0FederationConsole.png)

## SAML 2.0 Federation

- We need to setup trust between AWS [[iam]] and [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]] (both ways)
- [[AWS_SA_PRO_Obsidian_Notes/Master/SAML|SAML]] 2.0 enabled web based, cross domain SSO
- Uses the [[sts]] API: `AssumeRoleWithSAML`
- It is the old way of doing federation, recommended way by AWS is to use **Amazon Single Sign On (SSO)**
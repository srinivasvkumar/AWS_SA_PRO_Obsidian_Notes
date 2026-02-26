
---
tags: [management, governance, security, service:organizations, domain:deploy-management, concept:multi-account, concept:external-id, concept:control-tower]
---

1) Implementing [[control-tower|AWS Control Tower]] provides a centralized governance framework for managing multiple AWS accounts.
2) At times, you need to give a third-party access to your AWS resources (delegate access). One important aspect of this scenario is the _External ID_, optional information that you can use in an [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|IAM]] role trust policy to designate who can assume the role. 
	To require that the third party provides an external ID when assuming a role, update the role's trust policy with the external ID of your choice.
	To provide an external ID when you assume a role, use the AWS CLI or AWS API to assume that role.

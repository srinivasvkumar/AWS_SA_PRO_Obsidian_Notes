
---
tags: [security, management, governance, service:scp, domain:advanced-security-config, concept:permission-boundary, concept:organizational-unit, concept:policy-inheritance]
---

1) Permissions boundaries apply to user accounts but SCPs apply to entire AWS accounts and will be easier to enforce for all users.
2) Deny at higher level overrides allow (Deny -> Allow -> Deny)
3) You cannot modify an SCP that is applied to multiple OUs at one child OU. It must be edited in one place.
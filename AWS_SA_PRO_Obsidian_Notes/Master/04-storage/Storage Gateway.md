
---
tags: [storage, hybrid, service:storage-gateway, domain:storage, concept:file-gateway, concept:volume-gateway, concept:smb, concept:iscsi]
---

1) When using a file gateway the data is accessible via SMB locally in the on-premises data and is synchronized to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]]. Therefore, there the data is not accessible in the AWS cloud on a file system.
2) A [[storage-gateway|volume gateway]] is used for block-based storage not file-based storage. You access a [[storage-gateway|volume gateway]] over iSCSI rather than SMB.
3) 

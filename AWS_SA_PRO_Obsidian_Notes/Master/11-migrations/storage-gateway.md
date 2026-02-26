---
aliases:
- "File Mode"
- "Storage Gateway"
- "Tape - VTL Mode"
- "Volume Gateway"
---

# [[Storage Gateway]]

- Normally runs as a VM on-premises (or hardware appliance)
- Acts as bridge between storage that exists on-premises and AWS
- Presents storage using iSCSI, NFS or SMB
- On AWS integrates with [[ebs]], [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] and [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]]
- Storage gateways is used for migrations, extensions, storage tiering, [[dr]] and replacement of backup systems

## Volume Gateway

- Offers 2 different types of operation:
    - Volume Stored Mode:
        - The virtual appliance presents volumes over iSCSI to servers running on-premises (similar to what NAS/SAN hardware would)
        - Servers can create files systems on top of these volumes and use it in a normal way
        - These volumes consume capacity on-premises
        - [[Storage Gateway]] has local storage, used as primary storage, everything is stored locally
        - Upload buffer: any data written to the local storage is also copied in the upload buffer and it will be uploaded to the cloud asynchronously via the [[Storage Gateway]] endpoint
        - The upload data is copied into [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] as [[ebs]] snapshots which can be converted into [[ebs]] volumes
        - It is great to do full disk backups, offering excellent RTO and RPO values
        - Volume Stored Mode does not allow extending the data center capacity! The full copy of the data is stored locally
        ![Volume Stored Mode architecture](AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/images/StorageGatewayVolumeStored.png)
    - Volume Cached Mode:
        - Volume Cached Mode shares the same basic architecture with Stored Mode
        - The main location of data is no longer on-premises, it is on AWS [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]]
        - It has a local cache for the data only storing the frequently accessed data, the primary data will be in [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]]
        - The data will be stored in AWS managed area of [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]], meaning it wont be visible using the AWS console. It can be viewed from the [[Storage Gateway]] console
        - The data is stored in raw block state
        - We can create [[ebs]] volumes out of the data
        - Volume Cached Mode allows for an architecture know as data center extension
        ![Volume Cached Mode architecture](AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/images/StorageGatewayVolumeCached.png)

## Tape - VTL Mode

- VTL - Virtual Tape Library
- Examples of tape backups: LTO-9 (Linear Tape Open) Media which can hold 24TB raw data per tape
- Tape Loader (Robot): robot arm can insert/remove/swap tapes
- A Library is 1 ore more drives, 1 or more loaders and slots
- Traditional tape backup architecture:
    ![Traditional tape backup architecture](AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/images/TraditionalTapeBackup.png)
- [[Storage Gateway]] Tape (VTL) Mode architecture:
    ![Storage Gateway Tape (VTL) Mode architecture](AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/images/StorageGatewayVTL.png)
- A Virtual tape can be from 100 GiB to 5 TiB
- A [[Storage Gateway]] can handle at max 1PB ot data across 1500 virtual tapes
- When virtual tapes are not used, they can be exported in the backup software marking them not being in the library (equivalent of ejecting them and moving them to the offsite storage)
- When exported, the virtual tape is archived in the Virtual Shelf which is backed by [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Glacier]]
- [[Storage Gateway]] in VTL Mode pretends to be a iSCSI tape library, tape change and iSCSI drive (physical tape backup system)
- Use cases: 
    - On-premises data storage extension into AWS
    - Migration of historical sets a tape backups

## File Mode

- [[Storage Gateway]] manages files in File Mode
- File Gateway bridges on-premises file storage and [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]]
- With File Gateway we create one or more mount points (shares) available via NFS or SMB
- File Gateways maps directly onto on [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] bucket above which we have visibility from the AWS console
- File Mode uses Read an Write [[api-gateway|Caching]] ensuring LAN-like performance
- File Gateway architecture:
    ![File Gateway architecture](AWS_SA_PRO_Obsidian_Notes/Master/11-migrations/images/StorageGatewayFile.png)
- For Windows environments we can use AD [[api-gateway|authentication]] to access the File Gateway
- File Mode can be used for multiple contributors (multiple shares on-premises)
- File paths in a File Gateway map directly to [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] object names
- `NotifyWhenUploaded`: API to notify other gateways when objects are changed
- File Gateway does not support any kind of object locking => one gateway can override files from another gateway. We should use a read only mode on other shares or tightly control file access
- The bucket backing the File Gateway can be used with cross-region replication (CRR)
- The lifecycle [[policies]] can also be used for files to be moved automatically between classes
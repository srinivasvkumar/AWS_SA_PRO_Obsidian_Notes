## Advanced Architecture

At its core, the [[Storage Gateway]] Tape Gateway is a virtual tape library (VTL) that uses the iSCSI protocol to connect to physical tape drives or tape libraries. The Tape Gateway supports linear tape file system (LTFS) format, which allows for file-based access to data stored on tapes. This means that data stored on tapes can be accessed as if it were files in a file system, enabling easier migration, management, and retrieval of data.

The following diagram shows an example of a Tape Gateway architecture at scale:

```mermaid
graph LR
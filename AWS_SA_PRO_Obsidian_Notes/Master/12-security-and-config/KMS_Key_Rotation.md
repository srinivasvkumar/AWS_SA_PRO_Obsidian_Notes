## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, [[kms]] (Key Management Service) is a fully managed service that covers generating, storing, and managing cryptographic keys. It offers fine-grained access control for these keys and supports various encryption modes. Understanding [[kms]] [[RDS_Instance_Types|internals]] involves diving into the concepts of Key Material, Customer Master Keys (CMKs), and Key [[policies]].

### Key Material
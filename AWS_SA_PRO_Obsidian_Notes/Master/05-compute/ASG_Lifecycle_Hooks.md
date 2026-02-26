### [[RDS_Instance_Types|1. Advanced Architecture]]

[[asg]] [[asg|Lifecycle Hooks]] operate by allowing you to specify actions before or after an instance is launched, terminated, or recycled within an Auto Scaling Group ([[asg]]). This is achieved through two types of hooks: `ApplicationStart`/`InstanceTerminated`/`InstanceLaunched`/`InstanceFailedLaunch`/`InstanceFailedTermination`
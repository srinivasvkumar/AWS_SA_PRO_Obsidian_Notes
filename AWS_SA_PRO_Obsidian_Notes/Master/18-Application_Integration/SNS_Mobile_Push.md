## Advanced Architecture

At its core, Amazon [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Simple Notification Service (SNS)]] Mobile Push is a platform that enables sending push notifications to mobile devices using various services such as Amazon Device Messaging (ADM), Apple Push Notification service (APNs), Firebase Cloud Messaging (FCM), and Windows Notification Service (WNS). The following diagram shows an advanced architecture of [[sns]] Mobile Push.

```mermaid
graph LR
    A[Application] -- Publish --> B([[SNS]] Topic)
    B -- Subscribe --> C[Mobile App]
    B -- Subscribe --> D(Email/SMS/...)
    E[Notification Service] -- Store/Forward --> F(Target Endpoint)
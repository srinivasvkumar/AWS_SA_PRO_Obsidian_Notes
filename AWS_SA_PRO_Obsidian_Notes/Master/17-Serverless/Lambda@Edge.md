
1) Lambda@Edge lets you run Node.js and Python [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] to customize content that [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] delivers

2) [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] to change [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] requests and responses at the following points:

• After [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] receives a request from a viewer (viewer request)
• Before [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] forwards the request to the origin (origin request)
• After [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] receives the response from the origin (origin response)
• Before [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|CloudFront]] forwards the response to the viewer (viewer response)
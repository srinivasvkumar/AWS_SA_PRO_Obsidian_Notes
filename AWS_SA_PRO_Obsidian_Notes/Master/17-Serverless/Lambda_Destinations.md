## [[RDS_Instance_Types|1. Advanced Architecture]]

At its core, [[lambda]] Destinations is an event routing service that allows you to direct responses from asynchronous [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|Lambda functions]] to [[Master/Git_hub_notes/AWS-SAP-C02-Notes-main/README|other AWS services]]. This is achieved using two key components: **[[lambda]] function destinations** and **event source [[cloudformation|mappings]]**.

### Function Destinations

Function destinations enable automatic invocation of another [[lambda]] function when the initial function completes. They support various triggers such as:

- Success/failure of the initial [[lambda]] function
- Batch item failure or success in case of asynchronous invocations
- Specific values in response payload

Destinations can be configured within the [[lambda]] console, AWS SDKs, or [[cloudformation]] templates. For example, if you want to process successful image uploads to Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] by creating corresponding records in Amazon [[dynamodb]], your destination configuration would look like this:

```yaml
Type: LambdaFunction
Properties:
  FunctionArn: !GetAtt UploadProcessorFunction.Arn
  InvocationType: RequestResponse
  Qualifier: $LATEST
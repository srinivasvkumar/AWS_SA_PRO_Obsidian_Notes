---
aliases:
- "AWS SAM - Serverless Application Model"
---

# AWS SAM - Serverless Application Model

- A serverless application is not just a [[lambda]] function, it can include many more services such as:
    - Front end code and assets hosted by [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]] and [[cloudfront]]
    - API endpoint - [[api-gateway|API Gateway]]
    - Compute - [[lambda]]
    - Database - [[dynamodb]]
    - Event sources, permissions and more...
- The SAM group of products and features within AWS has 2 main parts:
    - AWS SAM template specification, which is an extension of [[cloudformation]]. Adds components to [[cloudformation]] templates designed specifically for serverless applications: Transforms, Globals & Resources (can include normal CFN resources as well as SAM specific ones)
    - AWS SAM CLI allows local testing, local invocation and deployments into AWS
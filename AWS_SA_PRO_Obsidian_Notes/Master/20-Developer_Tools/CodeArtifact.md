### Advanced Architecture

At its core, AWS CodeArtifact is a fully managed [[Artifact]] repository service that can be used to store various types of packages including JavaScript (npm), Java (Maven), .NET (NuGet), Python (PyPI), and Docker images. It integrates seamlessly with AWS services like CodeBuild and [[cloudformation]].

Internally, CodeArtifact uses Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/S3|S3]] for storing packages in highly durable [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|storage classes]] across multiple regions. Domain names serve as namespace for your packages, while upstream repositories allow you to proxy external package registries such as npmjs or maven central.

Here's a Mermaid diagram illustrating how CodeArtifact works under the hood:

```mermaid
graph LR
A[CodeArtifact Repository] --> B(Amazon [[Srinivas_Notes/S3|S3]])
C[Upstream Repository] -->|proxies| A
D[Package Consumer] --> A

1) CodePipeline builds, tests, and deploys your code every time there is a code change, based on the release process models you define.
2) You can use [[cicd|AWS CodeBuild]] to both build and test code. CodeBuild can be configured with custom scripts to run tests and the result of the tests can determine the subsequent actions such as proceeding to deployment.
3) CodeDeploy is used for deploying the updates but it is not used for running testing scripts.
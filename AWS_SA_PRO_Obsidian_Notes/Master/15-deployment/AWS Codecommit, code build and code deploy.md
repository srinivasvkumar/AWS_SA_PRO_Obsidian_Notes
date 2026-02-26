
![[Screenshot 2026-02-12 at 9.06.36 PM.png]]


2) Code deploy: 

It allows the new version of the application to be deployed alongside the old version, ensuring minimal downtime. Traffic is gradually shifted from the old version to the new version, allowing for monitoring and quick rollback if needed.

This approach is particularly suitable for applications where reliability and quick recovery are crucial, such as financial services applications. CodeDeploy's built-in rollback capabilities provide an additional safety net, ensuring that the application can quickly revert to the previous stable version if the new deployment introduces any critical issues.

CodeDeploy configuration is used for in-place and blue/green migrations but not for canary releases.
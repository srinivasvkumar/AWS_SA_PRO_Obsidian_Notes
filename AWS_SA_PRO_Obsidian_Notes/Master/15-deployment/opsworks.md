---
aliases:
- "AWS OpsWorks"
- "Opsworks Mode"
---

# AWS OpsWorks

- [[AWS_SA_PRO_Obsidian_Notes/Master/Management_and_Governance/OpsWorks|OpsWorks]] is configuration managed service which provides AWS managed implementation of Chef a Puppet
- [[AWS_SA_PRO_Obsidian_Notes/Master/Management_and_Governance/OpsWorks|OpsWorks]] functions in one of 3 modes:
    - Puppet Enterprise: we can create an AWS Managed Puppet Master Server (desired state architecture)
    - Chef Automate: we can create AWS Managed Chef Servers (similar as IaC, set of steps using Ruby)
    - [[AWS_SA_PRO_Obsidian_Notes/Master/Management_and_Governance/OpsWorks|OpsWorks]]: AWS implementation of Chef, no servers. Chef at a basic level, little admin overhead
- Generally we should only chose to use them if we are required to use Chef or Puppet, for example in case of a migration
- Other use case would be a requirement to automate
- If you see any mention of Recipes, Cookbook or Manifests than you know that it would be any of the three options mentioned above.

## Opsworks Mode

- **[[cloudformation|Stacks]]**: core components of [[AWS_SA_PRO_Obsidian_Notes/Master/Management_and_Governance/OpsWorks|OpsWorks]], container of resource similar to CFN [[cloudformation|stacks]]
- **Layers**: represent a specific function in a stack, example layer of load balancers, layer of database, layer of [[ec2]] instances running a web application
- **Recipes** and **Cookbooks**: they are applied to layers. We can use them to install packages, deploy applications, run scripts, perform reconfigurations. Cookbooks are collections of recipes which can be stored on GitHub
- **Lifecycle Events**: special events which run on a layer, examples:
    - Setup
    - Configure: generally executed when instances are removed or added to the stack. It will run on all instances, including already existing ones
    - Deploy
    - Undeploy
    - Shutdown
- **Instances**: compute instances ([[ec2]] instances or on-premise servers). They can be:
    - **24/7** instances: started manually
    - **Time-Based** instances: configured to start and stop on a schedule
    - **Load-Based** instances: turn on or off based on system metrics (similar to [[asg]])
- Instance **auto-healing**: [[AWS_SA_PRO_Obsidian_Notes/Master/Management_and_Governance/OpsWorks|OpsWorks]] automatically restarts instances in they fail for some reason
- **Apps**: they can be stored in repositories such as [[AWS_SA_PRO_Obsidian_Notes/Master/04-storage/s3|s3]]. Each app is represented by an [[AWS_SA_PRO_Obsidian_Notes/Master/Management_and_Governance/OpsWorks|OpsWorks]] App which specifies the application type and containing any information needed to deploy the app from repositories to instances.
- [[AWS_SA_PRO_Obsidian_Notes/Master/Management_and_Governance/OpsWorks|OpsWorks]] architecture:
    ![OpsWorks architecture](AWS_SA_PRO_Obsidian_Notes/Master/15-deployment/images/AWSOpsWorks.png)
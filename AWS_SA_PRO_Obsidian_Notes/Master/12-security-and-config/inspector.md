---
aliases:
- "AWS Inspector"
---

# AWS Inspector

- Is a product designed to check [[ec2]] instances and the operating systems running on those instances, as well as container workflows, for any vulnerabilities or deviations against best practice
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Inspector|Inspector]] can be run for a certain period of time (15 min, 1 hour, 1 day, etc.) to identify any unusual traffic and configurations which can put instances to risk
- At the end of this process, [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Inspector|Inspector]] provides a report of findings ordered by severity
- [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Inspector|Inspector]] can work with 2 main type of assessments:
    - Network Assessment: can be conducted agentless, but adding an agent can provide a more richer assessment
    - Network and Host Assessment: requires an agent to be installed. The host assessment looks for OS level vulnerabilities, so it requires the presence of an agent
- Rules packages: determine what is checked on an instance
- Examples of rule packages:
    - **Network Reachability**: 
        - Can be done with no agent or with an agent providing OS visibility
        - Checks reachability end to end
        - Returns the following findings:
            - `RecognizedPortWithListener`
            - `RecognizedPortNoListener`
            - `RecognizedPortNoAgent`
            - `UnrecognizedPortWithListener`
    - **Host Assessment**:
        - Agent is required
        - Checks for Common vulnerabilities and exposures (CVE)
        - Center for Internet [[appsync|Security]] (CIS) Benchmarks
        - [[appsync|Security]] [[Best Practices]] for Amazon [[AWS_SA_PRO_Obsidian_Notes/Master/Security/Inspector|Inspector]]
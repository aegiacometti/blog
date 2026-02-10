---
title: "From NetDevOps to SecDevOps - Continuous Compliance"
date: 2024-01-24
categories: 
  - "uncategorized-en"
tags: 
  - "cloud-function"
  - "continuous-compliance"
  - "devops"
  - "gcp"
  - "netdevops"
  - "netops"
  - "network-automation"
  - "python"
  - "secdevops"
---

Once NetDevOps becomes a part of your DNA, you will surely have some code or techniques to mix again and create new features that will significantly enhance your operations, just like this one.

On this occasion, the image speak for itself.

![](/images/WH-Continuous-Compliance-looker.png)

Every week, I receive a detailed report and a Jira ticket outlining necessary fixes for any deviations. This allows me to monitor the overall status, prioritize the top 10 deviations in each category, and track their progress over time. We're just beginning with this methodology, but our aim is to expand it by adding more checks and sites.

Here are a few key advantages of adopting this approach:

1. The impressive figure of 5,456 checks (by now) is **achievable only through automation**.
    
2. Shortened and merged process of creating standards and audit templates, as both are now represented as YAML files in a "**as Code**" format.
    
3. Standards and audits are treated equally. A mistake is a mistake, **ensuring clean and consistent setups** across the infrastructure.
    
4. By integrating Git for code management, Jira for ticketing, Looker for reporting, and Slack for communication, we achieve **end-to-end traceability**.
    
5. The reports are straightforward to understand and follow, with the ability to easily adjust the timeframe for **historical analysis**.
    
6. The Top 10 deviations per category provide clear **focus areas for improvement**.
    
7. **Configuration drift concerns are alleviated**, as any issues are automatically reported and assigned a Jira ticket for resolution.
    
8. The methodology allows for standard updates and using **reports as a guideline for changes**.
    
9. Post-deployment of new sites or devices doesn't require technicians to be fully updated with the latest standards. The automated audit runs as scheduled, generating reports and tickets for any discrepancies. This **reduces the skillset burden** on the technicians deploying the equipment.
    

**How to make this happen?**

The idea is pretty simple.

First, develop a script that periodically retrieves and converts the devices configurations into YAML format. Focus on retaining the crucial keys needed for standardization or auditing. YAML is the chosen format due to its readability, near-full compatibility with JSON, and support for comments.

Make that code run once a week and send the results from GitX to Slack (in our case) to start seing the changes in an MR format.

![](/images/merge.png)

Then copy that **standard.yaml** to **audit.yaml** and remove the keys that are not important compliance purposes. For instance, a networkID might be important for tracking but not necessary for auditing.

Next, write a second script designed to compare the YAML configuration of each device against the **audit.yaml** file.

This script will be also gathering the information needed for reports and generating tickets.

Finally, automate both scripts by scheduling their execution in a Cloud Function. 

This setup ensures regular and systematic checks without manual intervention and at a very low cost.

As I said before, NetDevOps is a one way road... Happy automating!

Thanks for reading.

Adrián.-

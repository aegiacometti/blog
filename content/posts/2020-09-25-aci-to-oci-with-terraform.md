---
title: "Cloud migration in 8 minutes: security rules from on-premise Cisco ACI to Oracle Cloud OCI with Terraform"
date: 2020-09-25
categories: 
  - "uncategorized-en"
tags: 
  - "aci"
  - "cisco"
  - "cisco-aci"
  - "cloud-migration"
  - "netdevops"
  - "oracle-cloud"
  - "oracle-oci"
  - "python"
  - "terraform"
---

**Introduction**

A couple of months ago I was tasked to accelerate a cloud migration from the networking perspective.

Context:

- Cisco ACI already deployed, with security rules grouped by functions and already agreed with the security team
- Migration to Oracle Cloud (aka OCI) already started and growing

But, since Cisco ACI and Oracle OCI are configured in a different way, now there were two growing and divergent schemas, where manual deployment on OCI would take several months and a lot of conflicts, between operations and delivery.

The concrete message was “Deployment of new VMs at OCI is being done fast with a Jira-Jenkins-Terraform stack, but networking and security is taking too long.”

So I thought it would be ideal to migrate/integrate the current and already well known security policy model from ACI to OCI, add networking pipelines, and integrate them with the current software stack for VMs deployments. I know, this sounds like one of those Sci-Fi PPT we all know... But why not? After all having Software Defined solutions on both ends has to pay back right? Let's try...

And it worked!

**Development**

Some short background first.

Managing and deploying security rules has always been tedious work. It has evolved drastically through time from being centralized in firewalls to being decentralized to the edge, from simple line to line configuration to complex SDN solutions.

So, back to the actual scenario.

The 1st task was to study their data structures and find a way to programmatically make a data structure translation, which in the end has to be deployed with Terraform.

Next, I will provide a very short and context-wise description of how security policies are implemented in Cisco ACI and Oracle OCI:

- Cisco ACI manages the grouping of servers with a construct named EPG, where static and dynamic server ports can be grouped. The grouping of security rules is a construct named Contract, which inside has another construct named Filter. Each Filter has several TCP/UDP/ICMP ports. The Contracts have several groups of Filters. An EPG consume or provide this contract, which means it allows ingress or egress traffic on those ports defined by the Filter. And finally the policy will be deployed at the network ports. It is a totally object reusable structure. Check this [link](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/1-x/aci-fundamentals/b_ACI-Fundamentals/b_ACI-Fundamentals_chapter_010001.html) for more details.

- Oracle OCI uses 3 constructs. Security Lists (SL), Network Security Groups (NSG), and Security Rules (SR - which are simple lines with source, destination, port, and direction). SL and NSG have limits on the amount of Security Rules they have inside. SL is applied only to subnets, hence their Security Rules use CIDR blocks. NSGs are associated to the vNIC of the VM, so the SR applies to a logical ID of the VM vNIC, making the NSG super scalable. Now instead of using the concept of providers and consumers, it uses the direction of the traffic (ingress or egress) which, in the end, is the same idea. Finally the policies will be deployed in the form of ipTables on the host where the VM is running. Check this [link](https://docs.cloud.oracle.com/en-us/iaas/Content/Network/Concepts/permissions.htm) for more details.

Keeping in mind that ACI can apply the policies from a networking point of view, and servers/cloud solutions have to do the same but from their own perspective, I quickly spot the common patterns (After all, we are always talking about some kind of software powered grouping of source IP, destination IP, and TCP ports).

In the following diagram you can see those patterns:

- EPGs are NSGs
- Filters in the Contracts are the plain Security Rules in the NSG
- and Provider/Consumer are Ingress/Egress

![aci-to-oci-model](/images/aci-oci.png)

So now the development (with Python) will be focused on:

- Download and walk the JSON configuration of a Cisco ACI Tenant
- Identify all the nested elements
- Generate an OCI like JSON dictionary representing the security rules from ACI
- Translated that JSON dictionary to Terraform files with Oracle OCI data structure.

As a convention I used EPG names as NSG names, and the Contracts/Filters definition will be applied directly as SR lines in the NSG.

Last but not least, the script should also identify and optimize the security rules, by ignoring invalid configurations (because of missing objects in ACI, normal after years in operation) and add new security policies.

**Conclusions** 

The outcome was pretty amazing:

- In just seconds the ACI configuration was downloaded, rules optimized, and Terraform/OCI files were created.
- From 4700 ACI rules, only 1700 were created at OCI.
- And those rules deployment only took 8 minutes (instead of several conflictive months)

In other words, in 8 minutes a full Cisco ACI Tenant security rules was migrated to Oracle OCI.

Fun fact: the ACI is in South America, OCI is in North America, and I’m in Asia at the moment with 500ms of latency. I’m sure those 8 minutes could have been 4 if I were closer!

Check out the video, see to believe right?

https://youtu.be/aHDO0EOuCHs

Code at: [https://github.com/aegiacometti/cloud-automation](https://github.com/aegiacometti/cloud-automation)

It's important to notice what has happened here: from a data structure containing all the configuration of an specific infrastructure, I had programmatically translated it to another data structure of a different kind of infrastructure, and deployed it in minutes.

This is a clear representation of the power in the mind-shift to Infrastructure as Code, as well as all the "as Code" terms going around out there.

Programatically we can insert an “if” to split the logic of the configuration in two parts and add in insert in the middle another device or function without losing their original logical intent.  

Following tasks will include:

- Since NSG is a relatively new OCI construct, we need to migrate the actual SL to NSG.
- Then create JSON Terraform templates for:

       - manage the CRUD of NSG/SL  
       - support of other appliances deployed at OCI like: firewalls and load balancers  
       - integration with Jira/Jenkins/Terraform pipelines

Don't hesitate in contacting me if you have any question or comment.

I hope you enjoyed the reading.

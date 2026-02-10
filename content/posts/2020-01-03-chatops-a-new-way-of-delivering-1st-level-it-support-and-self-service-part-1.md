---
title: "chatOps and automation: a new way of delivering 1st level IT support and Self-Service (part 1)"
date: 2020-01-03
categories: 
  - "uncategorised"
tags: 
  - "ansible"
  - "chatops"
  - "devops"
  - "free"
  - "netdevops"
  - "netops"
  - "network-automation"
  - "opensource"
  - "python"
  - "saltstack"
  - "slack"
  - "slackops"
---

This is getting extremely fun. After doing some testing with Ansible and Salt, the new integrant of the Netor family is [Slack](https://slack.com/intl/en-vn/). 

Yes, a chat platform, and I am going to show how much this can add to the game.

By adding some Python to parse the inputs and outputs from Slack and Ansible, it is possible to achieve some super practical things.

But wait, let’s first establish some basic concepts in this first part of the post, and then we will go to the good stuff in the next post.

What is automation about?

- Automation is not about replacing people
- It’s about automating daily boring and prone-to-error tasks that add no value to the staff, company, and customers
- which will, of course, end up being a high budget with low business value
- The goal of automation is to reorient technical staff to use that time to serve and support the real needs of customers and the business in general
- and finally, be faster and precise = efficient

Why we need it?

- Save time and money = wisely use of budget
- Reducing expenses in operation support and deployment
- Adding Self-service, to IT and non-IT staff for daily basic requests
- Potentially improve customer satisfaction
- Having scalability and massive deployment when required (vulnerability/critical fixes, viruses, etc)
- Automated compliance checks

How we do it?

- First by reducing the time of each activity to the minimal from minutes to seconds
- Automate simple tasks like user creation or status verification of a system
- Automate using Ansible/Salt which are market leaders in the automation field
- Where tasks are represented into Playbooks or States
- Eliminating basic support tasks and group them on playbooks/states
- Playbooks/states can contain tasks and/or scripts for any mix of operating systems
- If needed, create complex playbooks involving different systems and using a decision tree to trigger different actions in response to checks
- If needed, Create groups for massive deployment

Technically speaking, let's start with some basic activities per platform:

- Active Directory: create an account, enforce policies, disable an account, enable, lock, unlock, add and delete userID/PC membership from an AD group, view account status
- Windows/Linux: view system resource, install software, remote configuration, status and restart of services, network reachability from any to any kind of device (ping, traceroute and DNS)
- VMware/Hyper-V/Cloud: view system resources, reload VM
- Networking: view network status, routes, trace, reachability from any to any device, port statistics, links utilization, enable/disable ports, add/remove ACLs, verify the state of private links and VPNs, etc
- Automated compliance checks over all devices
- In seconds, distribute massively a new IT rule, as an example, to block a new virus

now let’s go to the great [second part](/posts/2020-01-19-chatops-a-new-way-of-delivering-1st-part-2/) of this post...

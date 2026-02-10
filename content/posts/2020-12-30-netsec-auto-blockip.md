---
title: "Network Security Automation Win - block IP threats in seconds"
date: 2020-12-30
categories: 
  - "uncategorized-en"
tags: 
  - "ansible"
  - "automation"
  - "chatbot"
  - "chatops"
  - "cisco-asa"
  - "devops"
  - "firepower"
  - "fmcapi"
  - "free"
  - "netdevops"
  - "netops"
  - "network-automation"
  - "python"
  - "secops"
  - "slack"
  - "slackops"
---

Not so long ago a customer asked me for help on how to block IP addresses on their regional firewalls infrastructure. They needed a solution urgently, since lately there were some severe attacks on other companies in the region, and they didn’t want to be “on the news” too.

Having a network engineer available 7x24 to block IP addresses on all the firewalls is not timewise efficient nor effective. It is simply not realistic to get him to work at any time (like midnight) and configure everything in less than 1 hour without errors and back and forths while chatting in the middle of the crisis with the rest of the organization.

I always keep in mind one of the biggest take-aways that I had from [Networking in Public Cloud Deployments](https://www.ipspace.net/PubCloud/) webinar from [ipspace.net](https://www.ipspace.net/).

It goes something like this (in my head): **_“Automate making my knowledge available to others who are NOT network engineers or even IT people, and if I can make that knowledge available on-demand, well.. that is a killer!”_** (huge win).

On the other hand, I like to use chatbots, because I think it is the simplest way to give a face to automation, and in the end, make my knowledge available on-demand. Check this [post](/posts/2020-02-10-the-new-team-member-a-bot-chatops-part-3/).

As a big plus, you have collaboration and searchable message history, which I like to see as a kind of knowledge base. Check this [post](/posts/2020-01-19-chatops-a-new-way-of-delivering-1st-part-2/).

You don’t need a huge platform, contracts, tech guys, customizations, web servers, databases, screens, GUIs, etc. It’s just on VM with free software all around, and a secure chat to interact with others or bots in this case.

So, let’s go back to the main topic and solve this for good, in an automated way, on-demand and it should run very fast (in just a couple of minutes).

Luckily the infrastructure was already standard on Cisco devices, traditional ASAs, and some Firepower managed by an FMC.

Since we were already doing network automation integrated with Slack chatbots (check this [post](/posts/2020-02-21-keep-it-simple-with-automation-bots/) with the architecture), this was relatively easy to accomplish: just develop a new chatbot command. It took me a couple of days to capture 80% of the infrastructure with Ansible, and some extra time to work with a public API to the FMC (thanks to [daxm](https://github.com/daxm) for the [fmcapi](https://github.com/daxm/fmcapi)).

In the end, any network or security support can block an IP in literally 2 minutes in the whole infrastructure (almost 30 firewalls). And, since it’s a Slack chatbot, he can do it anytime from anywhere with just a cell phone.

![](/images/blockip.png)

You might be thinking “isn’t that dangerous?” Yes, but there is command authorization and CIDRs checks included, so not everybody can block any IP.

Let’s go step by step on what is needed:

1.- Create a standard object name group, named “blacklist”. In that group we will add and remove IP addresses to be blocked.

2.- Only one time, create the ACL on the firewalls to block any IP which is in this “blacklist” group.

3.- To add IP/CIDR to be blocked in traditional ASA devices I used [this](https://github.com/aegiacometti/pychatops-basic/blob/master/ansible/playbooks/network/net-asa-obj-grp-msg-slack.yml) Ansible playbook with some Python scripts to parse the information.

4.- For Firepower devices I used FMCAPI. So in this case, I developed [this](https://github.com/aegiacometti/pychatops-basic/blob/master/ansible/playbooks/network/scripts/net-fmc-ip-to-networkgroups-msg-slack.py) Python script to do that.

5.- Add all kinds of controls in both points 3 and 4. For example, valid IP and CIDR, do not block internal IP/CIDRs, execution errors, active/passive cluster check, IP/CIDR already in the group, and of course, the last check is to verify that the outcome is how it's meant to be.

6.- You will write the IP/CIDR address to block in the Slack chat, the bot will read, validate, and execute the request, and post the output back to Slack.

7.- Everything is wrapped up with a daemon written in Python. Base code [here](https://github.com/aegiacometti/pychatops-basic).

Now we are working on how to integrate this functionality with [Splunk Phantom](https://www.splunk.com/en_us/software/splunk-security-orchestration-and-automation.html) (security automation and orchestration platform). In this way, it will be even more automated: when that platform detects a new threat in your infrastructure or an alert is externally pushed, it will automatically trigger the block IP command, in this case not represented by a chatbot, but using a direct API call to the bot daemon.

Hope you enjoyed it, until next time

Adrián.-

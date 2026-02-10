---
title: "Cheap, Fast, Easy and Secure with Automation Bots"
date: 2020-02-21
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

Automation Bots can help IT operations teams day-to-day work, and enhance the interaction between customers and IT members in a standard, fast, and secure way. _(Check the use cases from [my previous posts](https://adriangiacometti.net/), and new ones to come.)_

Just keep it simple!

That's the key to also enable easy adoption, a low learning curve, easy operability, and maintainability.

Let’s talk about those concepts.

**Cheap:** the software is [Open Source](https://opensource.com/resources/what-open-source): [Linux](https://en.wikipedia.org/wiki/Linux), [Python](https://www.python.org/), [Ansible](https://www.ansible.com/overview/how-ansible-works), [Salt](https://docs.saltstack.com/en/latest/), and [Slack](https://slack.com/intl/en-vn/features) (not Open Source but with a free account is enough). So it’s better than cheap, it’s free. I will talk more about this later.

**Fast:** having a response from a Bot for an automated action that will check the full network and server reachability in less than a minute. Well… that's fast, isn’t it?

**Easy:** let’s see the architecture of the solution.

Basically, what I’m doing here, is using Slack as a messaging interface between the Internet and your internal network, and Python as the mediator for getting those messages, take action and post back to Slack the results.

The “take action” part it’s also an interface to launch whatever OS command, in this case, interfacing with Ansible, Salt and other Python scripts (since they are multi-platform they support almost every OS, network, servers, etc).

The following diagram shows the interaction flow and directions:

![Automation Bots](/images/Topo-Demo-Lab-1.png)

The Automation Bots are daemonized Python scripts and managed by [systemd](https://en.wikipedia.org/wiki/Systemd). Which means easy control of the process.

If someone mentions the Bot name in Slack, then the Bot will parse that message, check user rights, permissions, validate the requested action and add it to a simple scheduler.

The Bot by using Ansible, Salt or a Python script, will ask the remote device to execute the action. The connectivity between the Bot and the remote device is achieved by using the standard methods provided by each device type. Let’s say: Windows - PowerShell, Networking and Linux - SSH and APIs.

Finally, the requested action at the remote device is also part of the standard ones they already have. Some examples: ping, traceroute, show run, conf t, dir, get-service, set-service, sudo service and so on per each device type.

The Bot script is a generic multi-function Bot. When systemd launch a Bot, it will load the specific parameters and available modules. That’s it.

![](/images/Bot-architecture-2.png)

The Automation Bots have a simple plugin architecture. So, adding new functionalities doesn’t require to modify the base code, just add the plugin to the Bot folder. The plugin is just one file and is also very simple.

![](/images/action_sample-1.png)

**What is needed to have this working?**

1. A Linux server with Python 3, Ansible and Salt installed. (3GB RAM, 2 vCPU, 20GB storage)
2. A list of devices to manage
3. UserID/password to manage those devices (basic privileges)
4. A Slack free account

Going back to the **easy** quality, a long explanation may be… but simple enough: checked!

**Security**, OK let’s do this! Component by component:

_Slack:_ 

- For user identity, besides the basic user/password, it supports two-factor authentication, SSO integrations, and even IP address ranges to permit.
- Each user has to be invited to a workspace by a workspace admin
- Granular user management in the paid version (examples like join/create/invite activities)

_Linux:_

- Harden the OS according to your security policy, assuming you have one, if not, it’s easy to find the best practices by searching on the Internet.
- Classic userID authorization to manage processes and files (Python daemons, Ansible, Salt, etc)

_Ansible:_

- UserID/password encrypted by [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
- [Playbooks](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html) protected by regular SO file system rights

_Python daemons and scripts:_

- UserID permission to talk to each Bot
- Bot authorization to talk on specific channel IDs
- Bot authorization to talk with specific userIDs (for SysAdmin tasks)
- Only outgoing traffic to Slack

_Logging:_

- Every process is logged: requested actions, authorizations, messages, outcomes, daemons, etc.
- And every log is [log rotated](https://en.wikipedia.org/wiki/Log_rotation)

_Networking:_

- If the Bots system is installed inside your premises, you only need outgoing HTTPS to Slack.
- If it's installed on the cloud, you will need a VPN and permit all the administrative ports (SSH, PowerShell, HTTPS/APIs) to all the managed devices. In which case maybe it's a good idea to use a reverse proxy.

I think **security** is pretty well covered by default.

**Easy adoption** is achieved by using Slack, a chat platform, everybody knows how to chat right? For SysAdmins, the Python code and Ansible playbooks are pretty readable and straightforward. Take into consideration that _I’m not a professional developer and it took me 3 months to get to this point._

Regarding the **Low learning curve**, from the end-user perspective, it’s just chatting with the Automation Bots and discover the available commands to ask for execution. For the SysAdmins, we are using well-known solutions. Python and Ansible are both very clear and there is plenty of documentation on the Internet.

**Slack is a key enabler for simplicity** since it makes possible to have a secure channel between the Internet and Intranet. You don’t have to worry about creating a messaging platform which could be a huge stopper. With the free account, you can do everything I’m showing here. The paid account add extra functionalities, like longer messaging history, creating buttons/forms, and a lot more, [check them on Slack](https://app.slack.com/plans/TQJ9F14QJ/). But if you want nicer things like buttons/forms, you will need a post-action from Slack to your Web server. So in this case, you also need a Web server to process those posts/forms and enable incoming traffic. I don’t especially like that part because it will add to the solution a component to manage not much necessary by now as buttons. I prefer to keep it simple and functional by now.

We all had the chance to deploy, install and maintain big tools with tons of functionalities, and then having a simple PC on the side to run Cacti. Ironic but true.

> Simple things have the potential to add more value than big tools, and _I’m not selling a product here, I’m sharing an idea which turned into a development._

Now, for the point of view of **operability** and **maintainability** (aka support), if you need corporate-like support for these tools, you can easily find it. Slack already includes it in the paid subscription. Ansible through Red Hat. Salt from Salt Stack. Linux and Python, well, being based in simple integration scripts you will find support easily. But if you need more holistic support, you can look for automation companies or individuals if you have a small company.

Just remember to keep it simple! Don’t buy a Ferrari to go for your first coffee. Don't try to build a rocket for new year’s eve fireworks (even if it sounds cool!).

#### **What’s next:**

- Develop a scheduler to prevent systems overload.
- I will publish the code for the basic usage and a demo to play with. But I have to admit that I’m a bit shy about this not being a pro in software development. With a little of Python skill, it’s easy to add new functionalities/plugins. The code and debugging are simple. It’s just about integrating standard inputs and outputs.
- Develop a Bot for quick monitoring. Useful for short-term while making changes on the infrastructure, or for long-term if you don’t already have a monitoring platform.
- Develop a Bot for changes approval.
- Add Salt to the Automation Bots. While Ansible is always agentless, Salt requires a software agent on the servers. For network devices is not strictly necessary. Anyway, Salt is a great tool and it will be part of this without any doubts. Besides, this will enable [event-automation](https://docs.saltstack.com/en/getstarted/event/) at the Bots level, and this is a big deal.
- Make the Automation Bots talk to each other.
- Add some sort of intelligence to the Automation Bots to extend the help with suggestions based on semantics or most common questions.

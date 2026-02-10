---
title: "Going deeper in Network Automation"
date: 2019-12-20
categories: 
  - "uncategorised"
tags: 
  - "books"
  - "learning"
  - "network-automation"
---

Event-Driven Network Automation and Orchestration topics can be addressed with Salt. It is the next step in this Automation revolution, where we are trying to make things faster and smarter.

- ![](/images/Untitled.png)
    

**Network Automation at Scale by Mircea Ulinic & Seth House**.  
This is a great book with a lot of examples, in only 95 pages goes through all the concepts and components. By the end it opened my mind to things that can be done today very easily, that is awesome. Great job by the authors!

Salt (which is Open Source) includes a native event-bus system and DB/cache, which means that it can react instantly to messages (ex Syslog, files changes, etc) in an automated, orchestrated, or event-driven way, and even can do it by taking in consideration the operational/on-going information already/constantly collected from the devices like MAC, ARP, BGP, etc. Adding to this the capability of having an active session with the devices, directly translates to high reaction speed.

Salt works in a Master/Slave configuration, where the Slaves are called “Minions”. The Minions are agents installed on the endpoints, which can be a downside, but you also have the option to run a Minion in a “Proxy” mode. In essence, the Minion as an agent is a process collecting information and keeping an open channel with the Master node (it also supports multi-master deploy for HA or zone distribution of load). Now, the “Proxy Minion”, is also a process, but this time that process is running detached from the target server, since it is a process it can run on another server and achieving the same objective by using a remote connection like ssh, https, etc. And lastly, since it is a simple process, you can run many Proxy Minion processes in a single server, it only depends on the available memory and CPU of that server.

Yes, you can run all the Proxy Minion processes for a site, on one Proxy Minion server without installing the agents on all of those endpoints.

This is especially useful for networking gears where you can not install software agents.

For other OSs, you can install the agent or use the Proxy Minion process mode.

Moving forward, the native capacity of reacting to events means that you don’t exclusively need an external monitoring system to react/trigger some automated action. It can be done in the same software without the need to start chaining, learning and administering platforms to achieve a simple automated event or reaction. Which in the end, as an example, you will have to do with Ansible.

The event-bus and the DB/cache works right after the installation, you don’t need to do anything to configure it. This is great, you don’t need to administer or configure the bus neither a DB, which are always associated as a burden when using other products.

Later you can, of course, inject events in the event-bus by using its agents or https/API, like sending an UP/DOWN from Nagios to the Salt event-bus to trigger some action.

I really like Ansible, it's a must-know and is where I started. It doesn’t need an agent installed in the endpoints, it is simple and it has a very high adoption, but it is too static. I mean, you need something else to run with it if you want to orchestrate or react (event-driven). In Salt, this is native from day 0.

Like other Open Source software, it has a big community and a lot of integration modules.

The modularity of this Software makes it possible to add your custom modules. It is very adaptable, but of course, you need to know some coding if you want to develop.

On the other hand, if you want to just use it, you can. If you follow this book is not so hard at all and there are plenty of tutorials on the internet.

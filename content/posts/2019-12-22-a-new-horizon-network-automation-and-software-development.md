---
title: "A New horizon: Network Automation and Software Development"
date: 2019-12-22
categories: 
  - "uncategorised"
tags: 
  - "learning"
  - "netor"
  - "network-automation"
  - "software-development"
---

Going into these new topics was one of the best decisions of the last years. It is an exciting and almost uncharted world.

There is an ecosystem of applications succeeding in the servers world, while networking is going behind but still very encouraged.

It is hard to think about the use cases without the tendency of self-making it too difficult. But starting with small things made me realize that the opposite way is the right way. No company is the same, not a single context is the same, so the point of view and development is always individualized. Start with small automation, and it will grow as time passes by.

My first step was learning Python, I knew some coding with other languages and even tried Ruby and Go, but in the end, I fell in with Python.

After that, I studied virtualization and SDN with Cisco ACI, VMWare, Openstack and Amazon. Soon I realized that at the base level, all of them look the same. The power of deploying servers, software packages, a full-stack application, and network rules in just minutes was simply undeniable. Because on the other hand, I was dealing with very slow (months) business deployments.

Now, since this is not magic, for managing this new kind of systems there is a lot of code to be understood.

That’s why I started to learn Ansible, and the first experience was great. In less than a month I finished the configuration standardization of hundreds of network devices in a region. I executed Ansible playbooks for 10 countries at the same time, and without the need of involving the network teams of each country. Doing this in an old fashion way, device by device, country by country would have been too expensive for the company in terms of hours invested/$$$ by the network operation teams. On the other hand, by investing a little time with Ansible, I was able to do the configuration changes and go back to contribute to business projects. Amazing! 

Later on, I decided to look for other options besides Ansible because by that time the network modules where very basics. And this is how I found Salt.

The power of managing systems with Salt is incredible, the concepts are slightly different, and you need to learn a little bit more of code. Anyway, this is a one-way road.

Salt has an event-bus and an internal database, that was the first interesting point because it allows you to do event-driven automation (when a message arrives at the bus you can trigger actions), and the second point was that definition of “states”. By which you code the desired state, and Salt will deploy it in any Operating System. In other words, you abstract basic configuration of NTP, SNMP, SLA, etc, using only the important and variable part like IPs, ports, communities, etc, and let Salt translate the rest of the configuration line internally with extension modules like NAPAL for networking, and others for other Operating System and applications. Basically, I managed to configure IP SLA in several routers at the same time and to enforce each night a set of basic configuration parameters like logging, SNMP, and users. Something interesting from the Security point of view, and might change after a day of operation.

At that moment, I decided to start building a platform, or just a compilation of software. The main objective was to integrate the functionality of Ansible and Salt, in one platform. Because both systems are good, and also both use different syntax and inventories. So this software that I was building was integrating the inventories, and masking complex command lines, in simple short lines/scripts. This is how the idea of [Netor](https://github.com/aegiacometti/netor) (Network Orchestra) was born, a self-learning project, by which I would learn more Python, Ansible, and Salt, and at the same time start with Git, pip, objects, unit-testing, and details about the underlying operating systems (Linux and macOS).

---
title: "The power of simplicity applied to monitoring"
date: 2021-07-26
categories: 
  - "uncategorized-en"
tags: 
  - "ansible"
  - "linux"
  - "monitoring"
  - "pymond"
  - "python"
  - "slackops"
  - "systemd"
  - "webhook"
---

Some time ago I was asked for help to develop a "simple and quick" Python script to monitor the status of a Linux service. They were in the middle of a deployment and they needed something fast.

They added, keep the result in a JSON file with the date, serve the directory with HTTP for quick view, and finally post it to ELK for graphics.

Ok, **this looks simple part by part, but the sum ends up in a very complete solution at the same time!**

Since simplicity was the key, the script has to do just that with the fewest lines of code, and I added that it shouldn't be necessary to install additional libraries or SDKs (aka dependencies).

This made me remind of the funny but true acronym KISS (Keep It Simple Stupid), well known in software development groups.

In a couple of hours, the script was ready, and in less than 100 lines of code, it was all included.

[https://github.com/aegiacometti/pymond/blob/master/pymond.py](https://github.com/aegiacometti/pymond/blob/master/pymond.py)

But this process made me think a lot about those huge monitoring platforms with which I had worked before, and I fell in love with the idea of keeping it extremely simple but fully functional.

Because, this script IS simple, has a small footprint, gives a lot of functionalities, and can be deployed to multiple servers with a simple Ansible playbook or any other automation tool (Chef, Puppet, etc).

But wait, to push a bit more I daemonized the script to be controlled by systemd (it could have been a simple cron task)

And lastly, to make it even better, I added ICMP checks, and send events to Slack with a webhook.

That's it! [**pymond**](https://github.com/aegiacometti/pymond) was born.

Everything in only 150 lines of code.

**Voilà a light but very complete monitoring solution.**

[https://github.com/aegiacometti/pymond/blob/master/pymond2.py](https://github.com/aegiacometti/pymond/blob/master/pymond2.py)

It is not hard at all to customize and monitor whatever you need in a server, and make it push the results to a collector, or even a slack message straight to you when there are problems. You only need to add your functions to the loop.

Today I'm using this script to monitor my workstation and a couple of servers.

I receive the alerts in Slack

![](/images/slack.jpg)

 and I can easily check all previous statuses

![](/images/web.jpg)

Very interesting, the power of simplicity!

Check the full repo for the configuration files to tweak:

[https://github.com/aegiacometti/pymond](https://github.com/aegiacometti/pymond)

Cheers

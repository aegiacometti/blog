---
title: "Free Local Network backup with GUI and messaging"
date: 2020-04-30
categories: 
  - "uncategorised"
tags: 
  - "ansible"
  - "chatops"
  - "devops"
  - "free"
  - "gitops"
  - "gogs"
  - "netdevops"
  - "netops"
  - "network-automation"
  - "network-backup"
  - "opensource"
  - "python"
  - "slack"
  - "slackops"
---

In my previous [post](/posts/2020-04-13-free-cloud-network-backup-with-gui-and-messaging/), [](/posts/2020-04-13-free-cloud-network-backup-with-gui-and-messaging/)I showed you how easy it is to have a network configuration backup with a GUI and messaging and keep it synchronized with GitHub to see the changes, and also sending messages to Slack and emails when it runs. For this post, I modified the scripts to use a cool Open Source tool named [Gogs](https://gogs.io/) (Thanks Nikolay Ryzhkov for bringing this tool to my attention).

With Gogs you will have a local repository system with the same functionalities as GitHub, so you don't need to worry about sending your configs outside of your own infrastructure.

We will use Gogs in a [docker container](https://cloud.google.com/containers), the installation and startup are incredibly fast and easy.

https://youtu.be/\_ncRyy5Spqk

You can find the code and configuration guide here: [https://github.com/aegiacometti/netconf-backup-gogs](https://github.com/aegiacometti/netconf-backup-gogs)

If you want to check how to create a repo follow this [link](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-new-repository).

Check the previous [post](/posts/2020-04-13-free-cloud-network-backup-with-gui-and-messaging/) to view all the benefits you can have with this setup.

Cheers.

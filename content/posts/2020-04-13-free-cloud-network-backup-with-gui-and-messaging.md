---
title: "Free Cloud Network backup with GUI and messaging"
date: 2020-04-13
categories: 
  - "uncategorised"
tags: 
  - "ansible"
  - "chatops"
  - "devops"
  - "free"
  - "github"
  - "gitops"
  - "netdevops"
  - "netops"
  - "network-automation"
  - "network-backup"
  - "opensource"
  - "python"
  - "slack"
  - "slackops"
---

Continuing with the development of my previous [post](https://adriangiacometti.net/index.php/2020/03/14/simple-network-configuration-backup/) where we made a very simple and complete solution for local network device configuration backups, including alerts via Slack or email. Now we are going to do some modifications to have a free complete GUI and messaging for Cloud Network configuration backups including the history, comments, and actionable buttons.

**_In the end you will be a simple click away to a full online historic track of all the configuration changes, including comments per line of changed code._** Take a look at this short video!

https://youtu.be/4YRJVrPvktQ

Basically, instead of having the backups in the local server, we are going to send them to a **private** GitHub repository and integrate it with Slack. This will give us some interesting benefits. For example not needing to worry anymore about the server/repository in terms of server maintenance (costs, licensing, monitoring, backups, space, availability, access, security, etc), or how to manage the messaging and history we need to have when something changes.

Yes, we will send the configurations to the cloud, the same as using other cloud services. Pay attention and care about confidentiality because you don't want your configurations to be public, and so make sure to set the repository as **Private**.

Later, you have to invite user per user to the Private repo. So you will use a master account as the owner and you will invite other as collaborators. This means that the master account shouldn't be used to work directly and should be protected by IT Security.

Two-factor authentication is supported in any account.

If you need to go deeper and be very granular about user rights in the repo, for $4 per month you can have an "Organization" account. If you want to use SAML Single Sing-On you have to pay a little more: $21 per month.

Anyway, having a secure backup solution with a GUI and messaging for free, or even $4 per month is ridiculously cheap in comparison to having to manage that server on your premises or anywhere.

Another benefit is having the configurations online and available 7x24 with a GUI to read and compare them!

You can find the code and configuration guide here: [https://github.com/aegiacometti/netconf-cloud-backup](https://github.com/aegiacometti/netconf-cloud-backup)

If you want to check how to create a repo follow this [link](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-new-repository).

In the next screenshots, I will show you the benefits I'm talking about.

## Slack

There are great benefits to interact via Slack with GitHub. So everybody is being notified and you can simply click in the message to go to GitHub directly.

1. View in Slack the configuration task run details, with messages like failed hosts (if there are any) and the finished network configuration backup. And the best, the message from GitHub with actionable messages.

![](/images/cloud-backup-tip7-1.png)

2. If you click in the Slack message "master" it will take you to the repository.

![](/images/cloud-backup-tip9.png)

3. If you click on "1 new commit" or the code number "xxxx" it will take you to the screen where you see the changes and you can add a comment regarding that change.

![](/images/cloud-backup-tip10.png)

## GitHub

More details about GitHub functionalities to follow your configuration changes.

1. When you enter the repository you will see that it's "Private", and you can see at once when the device configuration has been changed.

![](/images/cloud-backup-tip1.png)

2. If you click in the description with the message about the configuration change it will open a split window with the differences between the last backup and the new one. And there you can add a comment to that configuration change, maybe some reason, ticket number, etc

![](/images/cloud-backup-tip2-1.png)

3. Now going back to the first image, click on the device configuration file name. There are 2 interesting buttons: **Blame** and **History**.

![](/images/cloud-backup-tip3.png)

4. If you hit **Blame**, it will show the configuration changes by comparison.

![](/images/cloud-backup-tip4.png)

5. If you hit **History**, it will open a list with the configurations history in time, and with 2 other interesting buttons. The one on the left will take you to the same picture as the point 2 where you can review and add a comment.

![](/images/cloud-backup-tip5-1.png)

6. And the second button on the right will let you browse the repository at that exact time in history. In other words, you will see all the configuration files at that specific time. Check that both file comments are the same "first commit"

![](/images/cloud-backup-tip6.png)

Job is done! Awesome!

Now before continue reading take 5 minutes to process all of this...

Let's continue. This opens another chapter while getting close to what is known as "Network as a Code". There are some CI/CD integration tools, that would allow you to change the device configurations in GitHub and automatically deploy it on the devices. They support approvals, schedules, the full-blown of stuff we need to comply with. But... This is a BIG step into automation. Before going deeper, you should be completely comfortable about all the previous steps you read above.

Next steps: Triggering instant messages when a configuration change including who did it, and later move on to new things like GitLab as the CI/CD tool (Continuous Integration / Continuous Delivery).

I hope you enjoyed reading and don't hesitate to get in touch. Cheers!

---
title: "chatOps: a new way of delivering 1st level IT support and Self-Service (part 2)"
date: 2020-01-19
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

You don’t need a big budget to look smart and efficient. ChatOps and automation bots is easier than you tought.

**For who?**

This post could be helpful to any kind company and size: IT related and non-IT related, service provider, from small to large, the scenario is the same. The big difference with medium to large companies is that they have bigger budgets, and in this case, having the technology available to do this, do not actually involve big budgets, because it is free, it’s out there, it's [Open Source](https://en.wikipedia.org/wiki/Open_source). What you need to invest is just a bit in coding/service to adapt the software to your needs. It is not rocket science. I am going to show some basic things here in this post, and after it is only about “train/code” your bot by adding functionalities as times goes on.

**What to do?**

So, going back to the basics of the previous post, we need to quick test or resolve small daily tasks that take focus out of the main objectives of the company, customers and delivery of new products and services, or from another point of view, to easily continue your daily activities, business and non-business-related.

You don’t need to call someone anymore or filter those 200 email alerts, or to VPN and login to your monitoring console and jump servers, etc… just chat your bot and ask “him” to do something.

In seconds, you can have answers and perform tasks by chat on your computer or more interesting on your phone. Why is it so interesting on the phone? Because everybody has one all the time and everybody has adopted the chat in daily life.

**Why do this?**

To transform those low business value hours of basic support in hours of real business contribution.

We need to start thinking in automating the daily operational tasks and events to be faster, from anywhere, at any time = hugely more effective.

The industry is going deep into Artificial Intelligence and Machine Learning, so it is a good moment to start “training/coding” a bot to help us with basic daily tasks. So in the near future, your bot will be able to recommend to you how to perform activities based on semantics/Natural Language.

**How?**

Ok, now before moving to some **basic** use cases, I will ask you to stop 10 seconds on each one to think about it and keep in mind the difference in time/cost to perform these activities today, and any other possibility it might arise in your head for your own use cases. Pay attention to the different chatbot names as we go through the use cases and special attention to the time mark of each message! You will see what a fast answer means!

Let’s go to use cases:

- **Use Case 1:** Training the actual staff and each new person on the “how-to” procedures, or even at procedure changes, versus train the chatbot once, the output will be always the same:

![](/images/howto.png)

Each chatbot will have a general help about available commands:

![](/images/bot-help.png)

And also the specific help for each command:

![](/images/bot-help-extended.png)

- **Use case 2:** When you are not at your desk (having lunch, on weekends, at the movies, etc), and your boss is asking “is Internet OK?”, “the mail server is working?”, etc. What do you do?:

Option 1: scroll your emails, maybe call someone else, and run to have your computer connected to the company VPN and start checking, etc...

Option 2: just chat your bot and ask

Let’s see, imagine your boss (or someone else) is sending this to you (or to a support group chat):

![](/images/internet-theboss.png)

And just by clicking in “View Message”, you will be redirected to the detail of what he is talking about, what he asked to the chatbot…

![](/images/internet-theboss-extended-1.png)

This is a great and fast way of handling an issue right?

Now, as an expert with elevated rights, you can extend the command to get more info with “detailed=yes”.

![](/images/internet-detailed.png)

When there is an incident, this will save precious time by having a shareable and quick way of passing commentaries/tests between co-workers at the exact time you are working, at every level of expertise, and everyone can see it and refer to it.

- **Use case 3:** By using the search functionalities of Slack, the chat history in the channel could easily become evidence and a knowledge base for others to know how to proceed on similar events. You can even upload a file to the chat with instructions, logs, drawings, etc.

This will exponentially save time!

![](/images/all-search.png)

Now stop for a second, and think about this… We are running over a lot of traditional big, expensive and fat tools like knowledge base, change management, logging server, etc. I am not saying we don’t need them anymore, I’m saying that we are integrating functionalities and making them more accessible to everyone. This is not a replacement tool, it's a supporting tool, and of course, you have to belong to a certain group in order to have access to do certain things, just like today.

- **Use case 4:** Let’ see a use case with a cell-phone and with execute permissions. Security needs to block urgently an IP because of a new virus. You can do it in all the infrastructure in a couple of minutes. 

![](/images/chuck.jpg)

This kind of fast response could also help to fast answer an issue-related question from anywhere and let the flow of the incident management be rerouted to the next team if it isn't related to your area.

Let’s see a larger use case involving an incident and several teams.

- **Use case 5:** Now a use case with the full screen, for an incident.

Is the mail service working? (it could be any service, mail, web server, DHCP, etc)

- ![](/images/win-mail.png)
    

All of this is enabling us to go into Self-Service for IT and also for your different client departments. By allowing them to use this new approach on dedicated Slack channels, we could win in customer satisfaction (because of the quick and automated actions at their hands) and in our own “satisfaction”, because you will have fewer interruptions or resolve them faster, in the office and out of the office.

And for last, just a couple of the typical use cases:

- **Use case 6:** Is my account locked? (it could be also for unlocking it)

![](/images/derek-view.png)

- **Use case 7:** Or even disabling a userID in Active Directory (HHRR?!)

![](/images/derek-disable.png)

- **Use case 8:** Add/remove a userID from a domain group

![](/images/jhonny-remove-derek.png)

The use cases are infinite as your needs and creativity could be. Just keep in mind, easy and everyday repeatable tasks. That is why automation is customized for you and what your company needs. Pre-build tools can become very complicated to manage.

Lastly, you can use other Slack functionalities like creating dedicated support group channels, start group/channel voice calls! , predefined tasks and responses, create buttons, forms, calendars, schedules, personal notes, etc.

That is all for now. [In the next post](/posts/2020-02-10-the-new-team-member-a-bot-chatops-part-3/), I will explore more interesting use cases that will save a lot of time, and honestly, there is nothing in the market to do it so easy and handy. And of course, the security aspects of this approach. Don’t worry, security is everywhere.

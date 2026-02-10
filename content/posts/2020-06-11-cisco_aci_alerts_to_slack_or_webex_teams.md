---
title: "How Cisco ACI can easily send real-time notifications straight to your hands"
date: 2020-06-11
categories: 
  - "uncategorized-en"
tags: 
  - "ansible"
  - "chatops"
  - "cisco"
  - "free"
  - "netdevops"
  - "netops"
  - "network-automation"
  - "python"
  - "slack"
  - "slackops"
  - "webexteams"
---

Even if you don't fully understand what is an API, it is really easy to leverage their power to have some useful information directly sent to a collaboration platform and receive it on your mobile or PC in real-time.

All the tools in use are Free: Python, Slack, and WebEx Teams.

In this post, I will show you how to send Cisco ACI faults and events to messages straight to Slack and WebEx Teams.

With a Python script, we are going to subscribe to the ACI faults and events channel. Then when ACI has a new message it will send it back to the script, and the script will send that message to a collaboration platform like Slack or WebEx Teams. We could also be doing any other kind of action with that information.

Here in the first example, you can see how a **fault** will be seen on your PC, Cell Phone, or tablet, which means ... **_anywhere anytime!_**

Later if you add some chatBots you could easily take some action on these messages, forward them to support, start a voice call on the same chat, or even have them searchable for a future reference. Check my other posts about [chatBots](https://www.linkedin.com/pulse/chatops-new-way-delivering-1st-level-support-part-2-adrian-giacometti/).

_@Cisco WebEx Teams:_

![](/images/aci-fault-webex-teams.png)

_@Slack:_

![](/images/aci-fault-slack.png)

For the next one, I will upload a full tenant config to the APIC. You can see the EPGs creation **events**, and at the end, a **fault** because one of the interfaces does not exist. (I'm using the Cisco ACI Sandbox).

_@Cisco WebEx Teams:_

![](/images/aci-event-webex-teams.png)

_@Slack:_

![](/images/aci-event-slack.png)

Check [the code at GitHub](https://github.com/aegiacometti/cloud-automation/blob/master/aci_to_chat.py).

Now, this script is running on a server, and the Cisco ACI APIC, allows you to [deploy the script inside a small Docker container](https://www.cisco.com/c/en/us/td/docs/switches/datacenter/aci/apic/sw/2-x/App_Center/developer_guide/b_Cisco_ACI_App_Center_Developer_Guide/b_Cisco_ACI_App_Center_Developer_Guide_chapter_01.html) directly in the APIC. This is awesome because you are avoiding the need of a server to run the script, and **_we can say that the APIC is sending the message directly to you and without the need for any intermediary monitoring system._**

Stop 2 seconds, and think about this... The Cisco ACI API is just another API, we could be using Firepower API, AWS API, Microsoft Teams, WhatsApp, and so on. Even with Slack and WebEx Teams, we are using their APIs. Since all the APIs follow the same structures and methods, if you learn just a bit about how to "talk" to them, you will be able to "talk" with all of them, and in some time, make them "talk" to each other. Which is, in fact, what are we doing here, with some Python code perform a GET from an API for some info, and POST that info into another API. So many new possibilities right?

I hope you enjoyed reading. Don't hesitate to contact me if you need any help!

Have a good day, Adrián.-

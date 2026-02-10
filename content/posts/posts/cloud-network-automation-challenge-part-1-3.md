---
title: "Cloud & Network automation challenge: Deploy Security Rules in a DevOps/GitOps world with AWS, Terraform, GitLab CI, Slack, and Python (special guest FastAPI) - part 1/3"
date: 2021-04-08
categories: 
  - "uncategorized-en"
tags: 
  - "automation"
  - "aws"
  - "chatbot"
  - "chatops"
  - "cloud-automation"
  - "devops"
  - "fastapi"
  - "free"
  - "gitlab-ci"
  - "gitops"
  - "netdevops"
  - "netops"
  - "python"
  - "slack"
  - "slackops"
  - "terraform"
  - "webinar"
---

I found my next challenge while I was defining the Lab exercises for [Networking in Public Cloud Deployments](https://www.ipspace.net/PubCloud/) training course. Honestly, I didn't know much about what was ahead, but **I had a great learning idea and the will to go through the challenge**.

I'm not saying that this is correct nor complete, is my first run exploring the whole thing, so I'm just saying "hey! I can do this!, and **this is the Proof of Concept**.

Is all FREE". I will show you how I did it. Then your imagination is the next step.

This is the full architecture diagram, **the idea is to work with Security Groups in an Infrastructure as Code manner. Before deploy, the changes should be checked, notified and approved.**

![](/images/full-arq-diagram.png)

Note that since it is Infrastructure as Code it could be any other component or even the full configuration deployment. But for this PoC let's just stay with Security Groups.

I will explain some basic details very shortly, then some lines of code and of course some pictures. But is not easy, this is gonna be really wild (for me at least), we have so many new topics. Therefore, I will split it into 2 posts.

The repository for the code is [here](https://gitlab.com/aegiacometti/devops_cloud_challenge)[.](https://gitlab.com/aegiacometti/devops_cloud_challange)

Now a short abstract of what I will do with each technology:

 1.- AWS (VPC, EC2, S3, IAM, Security)

- Create a VPC with two subnets, front and back
- In EC2 deploy 2 servers, web front, and app back.
- Secure them with Security Groups for admin and public web access.
- Store a picture in an S3 bucket, and to simulate some data protection, this picture can ONLY be accessed by the web front public IP VM (this could be done with a VPC endpoint, but I don't want to keep piling new things)
- And do on your own some IAM to exercise users and roles without using the root account.

 2.- Terraform (with remote backend and basic provisioner)

- The Terraform states will be stored remotely at GitLab. It is not convenient to store them on a single server because if you lose them, you have a big problem for your IaC plans, basically, you just lost them :(
- I will use a basic Terraform provisioner to configure the WEB front VM at Linux Level and...

 3.- FastAPI

- Do my first FastAPI code and install a basic FastAPI web app
- FastAPI has to read and display the picture stored at the S3 bucket.

 4.- GitLAB CI (pipelines)

And here is where things will start to get really fun.

- I would like to use a pipeline to configure the AWS Security Rules to allow public access to the web front VM (at creation time I did it wrong on purpose so I can correct it in the second part in an IaC/GitOps manner)
- Validate if the requested rules are in compliance with some security standards
- Have a sort of manual approval
- And just then execute the changes with Terraform.
- Last but not least, **post-deploy tests to check if all my services are still running** (this point is VERY interesting too).

At the end of the story, we will have all the steps, logs, and configuration changes stored in a DevOps fashion at GitLab.

 5.- Slack

- Every step has to send a notification to Slack
- Include a compliance note and Terraform plan.
- The notification in Slack has to have a link to approve the deployment
- Another message when the deployment is done
- And finally the test results.

 6.- Python

And of course, my beloved Python will help me at every step to glue it all together.

Finally, the actual change request I will do straight by running the pipeline. Later you can modify it by just changing the IaC for the security groups into GitLab and commit it, or push it if you do it from your PC.

Since I have everything as IaC I could have done it beautifully with a Slack chatbot, but I already did that in previous posts and I wouldn't be adding anything else to this challenge besides repeating myself.

So, let's get started!

Requirements: accounts for AWS (we will use free tier only), GitLab and Slack (everything on the cloud), and locally your Linux PC (I'm using Virtual Box) with Terraform, AWS Cli, and Git. All is free, just remember to run Terraform destroy after each session.

[Link Part 2 - AWS, Terraform, FastAPI](https://adriangiacometti.net/index.php/2021/04/08/cloud-network-automation-challenge-part-2-3/)

[Link Part 3 - GitLAB CI, Slack, Python](https://adriangiacometti.net/index.php/2021/04/08/cloud-network-automation-challenge-part-3-3/)

Conclusions

Having everything as IaC is what is being done (or trying) nowadays. Even if the legacy infrastructure is not the same as working in the cloud, this concept can still be adapted up to some degree for legacy systems.

Once you are talking in code with your infrastructure, there is a world of possibilities where your imagination is the only limit. I have proved just that in this new adventure. I didn't know what was ahead, but with just a bit of coding background, willingness to learn, and years of experience in enterprise networking, you can dream, imagine and build anything you might need for your day-to-day or long-term necessities.

Don't limit yourself, go for more!

Thanks for reading ;)

Adrián.-

---
title: "Cloud & Network automation challenge: GitLab CI, Slack, and Python - part 3/3"
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
---

In the previous [post](/posts/2021-04-08-cloud-network-automation-challenge-part-2-3/) we set up the infrastructure at AWS and we stored the states at GitLab.

Now let's go for the most fun part, integrate everything.

We will modify a security group to allow HTTP access to the FastAPI app, and we will have all kinds of notifications and requests for approvals sent to Slack.

Every step will be logged and stored in the form of messages in Slack and in a DevOps fashion at GitLab: terraform states, artifacts (which are specifics output that you want to store), logs for each run of the pipeline, etc. This will give you incredible traceability of your IaC.  

Now in this post, we will be working only in the root folder of the repo.

4.- Slack

- Every step has to send a notification to Slack
- Include a compliance note and Terraform plan.
- The notification in Slack has to have a link to approve the deployment
- Another message when the deployment is done
- Some consistency check
- And finally, the post-deploy test results.

The creation of an App in Slack can be tricky, I will give you general steps here.

- create an account in Slack and login to your desktop
- go to Settings & Administration -> Manage Apps
- Build
- Create new App
- Give it a name and deploy it on your desktop
- Go to Incoming WebHooks, activate it, and create a WebHook to a channel. Take note of the webhook code.
- Go to Oauth & Permissions, activate it on your desktop and it will give you the Bot User OAuth Token. Take note of the code.

To allow GitLab to send you messages to Slack, you will need the Slack WebHook, and add it to your project repository `Settings -> Integrations -> Slack`.

There you will add the Slack WebHook and select for which events you want GitLab to send you a message, and to which channel that message should be sent.

Since the pipeline will run each time you change the code and push the commit, you should get a message like this in Slack.

![](/images/slack-commit-message.png)

Now, these are the full notification messages that we want to get in Slack: requests, approvals, deploy status, compliance, terraform plan, consistency check, and end of pipeline status.

![](/images/slack-full.png)

And last, the execution of the test child pipeline

![](/images/slack-full-test.png)

**5.- GitLAB CI (pipelines)**

- I would like to use a pipeline to configure the AWS Security Rules to allow public access to the web front VM.
- Validate if the requested rules are in compliance with some security standards
- Have a sort of manual approval
- And just then execute the changes with Terraform.
- Last but not least, post-deploy tests to check if all my services are still running (this point is very interesting too).

This will be a hard section, but I learned a lot.You should read a bit about **what is it pipeline**, this a good [tutorial](https://docs.gitlab.com/ee/ci/quick_start/) at GitLab.Also, you could read a bit about **CI/CD** (Continuous Improvement/Continuous Development or Delivery), it comes from Software Development and is a practice to cover the steps of the building, testing, and deployment.I'm not a master in Software Development, honestly is a bit longer to get all the concepts, so I will try to make a short abstract for you with the main ideas. 

- If you know **Ansible**, even though is a different thing, both are a chain of sequential tasks written in YAML, each task will perform some action and give some output.
- Pipelines can be executed manually when a push to the GitLab repo happens, which is the default. We will do a manual one here.
- **In this case of pipelines, each task will create a docker container image to execute what it needs to do and it will produce some output useful for the next task, and it will destroy/delete the container.**
- These containers will run in a **runner** which is just a host with docker on it. You can use a free runner provided by GitLab or you can use your own. In this PoC I don't need my own runner, I don't have big things to compile as it could be the case of a bigger software development project, neither some security restrictions that force me to keep my data within my premise.
- **Now the output of each task will be stored as an artifact for later use in the pipeline.** Consider that the origin of this artifact concept is in Software Development, and it could be a compiled code, a big file, a new docker container maybe, etc, which is not our case, we will have small files with Terraform plan and states, and some other information that I will need later in the pipeline.
- Each task in a pipeline can use a different container image, oriented to Python or Terraform or whatever you need. You just need to consider that the objective is rapid startup and execution of the task, so you don't need a full Linux image with GUI just to run a small curl or python script.
- The general pipeline config in our case is **.gitlab-ci.yml**. This is configurable but we don't need to worry about that now.
- If there is an error at any task or stage the pipeline will stop unless that error is handled inside the pipeline.
- Pipeline executions are NOT fast, but they will produce always the same output as this is the corner stop for software development. Consider that each task will deploy a container, that is why is slow.

The pipeline is fully commented, so you can read it now the get the big picture of what is going to happen. Check it at [./.gitlab-ci.yml](https://gitlab.com/aegiacometti/devops_cloud_challenge/-/blob/master/.gitlab-ci.yml) 

- In the beginning, we will have basic start variables, which are stored at the project repository in GitLab and you will see them with syntax like ${variable}. These variables will be used through the pipeline.

![](/images/pipeline-init.png)

To set up these variables in YOUR fork of the repository in GitLab, go to `Settings -> CI/CD -> Variables -> Expand -> Add variable`, and add these two variables. This will allow the pipeline to pass the values when you execute Terraform.

![](/images/gitlab-cicd-variables1.png)

- We will define stages where all the tasks will be distributed

![](/images/pipeline-stages1.png) This is the list of pipeline execution results with a small status, this data is updated in real-time, so is very useful to follow the execution. You can see it in GitLab at `CI/CD -> Pipelines` ![](/images/pipeline-graph0.png)

And if you click in one of the pipelines number you will see a graphical representation of it:

![](/images/pipeline-graph1.png) ![](/images/pipeline-graph2.png)

This last one is a very important stage, I will go deeper later. Basically, I'm injecting at the end an external **child** pipeline code to do some checks:

![](/images/pipeline-graph3.png) ![](/images/pipeline-graph4.png)

IMPORTANT to have in mind: Each task will trigger the creation of a container, with an image, import the project repository variables and execute what is specified, produce output, optionally store it as an artifact, and then the container will be destroyed.

Now, if you click in any of these stages and tasks you can see what happened inside in this typical Linux console style. This is the container running in the runner, and this information will remain stored, so this is part of the traceability.

![](/images/pipeline-runner1.png)

In the same screen, on the right, is where you can see the artifacts, they will remain stored and you can also check them.

![](/images/pipeline-runner1-output.png)

- Tasks and stages will have conditionals and different kind of controls
- Each task has a key-value "stage" to show to which stage it belongs

Let's go by grouping all the tasks per stage: _**5.1.- Stage Prepare:**_ to verify that all the environments that we will need can be created. Remember, this comes from Software Development, so it's important to be able to reproduce the environment at each stage independently.The script `gitlab-terraform.sh` is just a wrapper to avoid having big text lines in the middle of the pipeline, which only creates visual noise. (thanks to [Nicholas Klick](https://gitlab.com/nicholasklick)). In this case, I adapted to be able to produce text outputs instead of JSON. ![](/images/pipeline-prepare1.png)

_5.2.- Stage Validate:_

- 1st task **validate\_terraform** we validate that the terraform files have a good syntaxis.
- And for the 2nd task **validate\_compliance** I created a Python script that will check if the requirement is "compliant" with a security policy which for this PoC is defined inside the script, but it could be elsewhere. Check it at `./scripts/validate_compliance.py`
- This script will return exit(1)=error if the request is not compliant and so it will stop the pipeline or in this case jump straight to the stage that caches errors.
- Here we will store our first **artifact** (compliance.txt) generated by the script, which is useful as evidence and future reference.

![](/images/pipeline-validate1.png)

**_5.3.- Stage plan:_** we create the terraform plan and store it as an artifact for future reference in text format.

Note that I'm using terraform targeting, which is not recommended for everyday use, but for this PoC is a perfect fit.

![](/images/pipeline-plan1.png)

**_5.4.- Stage request approval:_** 

- in this task, we will send a message to Slack to notify that a pipeline has been launched
- we will include the compliance report
- the terraform plan
- and a link to approve or not the request

![](/images/pipeline-reqapp.png)

For this task you will need to add the Slack BOT USER OAUTH token in GitLab in the project CI/CD variables section like we did before

![](/images/pipeline-slack-oauth.png)

This is what you will get in Slack, the request notification, with the compliance report and the terraform plan. Also, the link to go to approve the request in GitLab in order to proceed with the changes.

![](/images/slack-deploy-request-approve.png)

_**5.5.- Stage deploy:**_ finally, if everything looks OK and the pipeline run has been approved, `terraform apply` will launch the changes to the IoC.

Note the line `when: manual` indicates just that, manual approval is required for this task, and combined with this other line is `allow_failure: false`, force the pipeline to stop any task until it is approved.

![](/images/pipeline-deploy.png)

To approve the pipeline you will need to go into GitLab, find the pipeline and the task to approve or use the link in Slack from the previous section, and hit the button play.

![](/images/pipeline-approve.png)

**_5.6.- Stage notify deploy:_** send the deploy notification message to Slack.

Notice the channel ID is specified in the curl command.

![](/images/pipeline-notifdeploy.png)

The message to Slack notifying the successful approval and deploy

![](/images/slack-approved-deployed.png)

**_5.7.- Stage verify deploy and notify:_** in this step, we could verify the integrity of what was executed. Now, this integrity check could get super complicated but is just to show what kind of automated control you could be doing in the script.

![](/images/pipeline-verifynotif.png)

Here you will need to add the last variable in GitLab CI/CD, to allow the pipeline to download the Terraform managed states in order to be analyzed by the python script. The user token is the same we got in the previous post for Terraform to upload the states.

You can find it in your notes, or if you don't, you can generate a new one at `GitLab -> user preferences -> access tokens`.

![](/images/pipeline-gitlab-ci-token.png)

In this case, the python script will print the repo terraform files, the terraform managed state files, and what is actually configured on AWS.

![](/images/slack-consistency.png)

**_5.8.- Stage notify if failure:_** is just a generic exit stage that will run only when another stage has failed. It will send to Slack the failed message with the link to the pipeline in GitLab for future reference.

![](/images/pipeline-onfailure.png)

**_5.9.- Stage check services:_**

**NOW, THIS IS GOLD !!!**

In this stage, we will INCLUDE an external **child pipeline** (what is inside is in the next section).

This enables us to create **reusable general networking TEST pipelines**, and instruct development teams to include them at the end of their pipelines.

In this way, when someone changes the infrastructure there is a standard test pipeline to include, and the result will be stored and published, which means not only alerting also traceability. 

![](/images/pipeline-checkpost.png)

6.- Reusable Test pipeline

This is a very simple and fictional pipeline, you should be able to read it now that we have covered the basics in the previous section.

Basically, it will have 3 stages: prepare, check\_services, and notify.

Check the child pipeline at `[./test-pipeline/verify-aws-services-gitlab-ci.yml](https://gitlab.com/aegiacometti/devops_cloud_challenge/-/blob/master/test-pipeline/verify-aws-services-gitlab-ci.yml)`

It will run a Python script that will do the following:

- download the terraform states from GitLab
- get from AWS the members of the public security group
- test HTTP get to check if the web page can be accessed for all members
- store the results as an artifact
- send the results to Slack

Check the python script at `[./test-pipeline/verify-aws-services.py](https://gitlab.com/aegiacometti/devops_cloud_challenge/-/blob/master/test-pipeline/verify_aws_services.py)`

The Slack message would be like this:

![](/images/slack-test1.png)

**7.- Running the pipeline**

Ok great! we are ready to run the pipeline.

To do that you can modify the code in your PC and push it to GitLab, or you can manually run the pipeline from GitLab, in this last case go to `CI/CD -> Pipelines` and it the button RUN pipeline, in the next window do the same.

This second window could be used to pass different variables in order to influence the pipeline in different ways, like users, versions, names, etc. But let's keep it simple to learn the fundamentals.

Now you should see the pipeline moving forward like in the previous screenshots. Be patient until it gets to the stage of Deploy, in which you will need to approve it as I already mentioned before.

You can follow the Slack message links.

Now this approval part is something that should be done with **git branches** and **merges**, is the right way to do it. But I would need a paid account in GitLab.

I know it will work, but I don't need that right now. I want to keep this free to try ;)

**Closing**

If everything went ok, now you should be able to see the full web pages served by fastAPI in the public IP of your front instance. If you don't remember the IP, you can check it at AWS or in your terraform state file under the key front\_public\_ip. 

![](/images/webpage.png)

WOW, this was super long and complicated, with many moving parts to learn. Once you get the idea, you will see that is not so crazy at all, but it takes time, it took me several months to internalize everything.

Any comments or suggestions or ideas to explore I will be happy to help!

Thanks for reading.

Adrián.-

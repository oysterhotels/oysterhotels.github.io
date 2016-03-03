---
title: Replacing crontab and Task Scheduler with Rundeck
author: Ben
layout: post
permalink: /replacing-crontab-and-task-scheduler-with-rundeck/
published: false
---

crontab on Linux and Task Scheduler on Windows are fine tools as far as they go. But they're very simple: difficult to schedule, no retrying, not centralized, you can't customize success or failure emails, almost impossible to get non-engineers to use, etc.

Enter [Rundeck](http://rundeck.org/). It's a powerful system, though we're only using it in "glorified crontab with a web UI" mode, which was a fine way to start. We've now switched most of our crontab jobs over to it, and we're working on moving our Windows Task Scheduler jobs as well. Here are the benefits:

## Decent UI with single-sign-on login

It has a decent UI that supports single sign on, so anyone on our team has automic access to it. This means our analytics team can write ETL scripts and set them up in Rundeck without involving engineering, saving everyone's time.

The UI is also good for engineering. It allows you to create jobs, run a job, view history, and check job output. Previously to look something up we'd have to hunt through old emails, or SSH/Remote Desktop into a machine and scan through log directories. Big plus.

The UI is definitely better than Windows Task Scheduler (not hard to beat), but I'm not going to say it's stellar. It has a very [made by developers](http://blog.codinghorror.com/this-is-what-happens-when-you-let-developers-create-ui/) feel, with tons of detail upfront, and some common operations like look at recent errors or log output hidden behind several clicks.


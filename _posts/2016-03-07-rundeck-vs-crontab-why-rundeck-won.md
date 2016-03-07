---
title: "Rundeck vs. Crontab: Why Rundeck won"
author: Ben
layout: post
permalink: /rundeck-vs-crontab-why-rundeck-won/
draft: true
---

We like automation here at Oyster: we do one-command [deployments using Ansible](http://tech.oyster.com/using-ansible-to-restore-developer-sanity/) and we run a lot of other automated scripts and ETL processing on various schedules.

crontab on Linux and Task Scheduler on Windows are fine tools as far as they go. But they're a little too simplistic: difficult to schedule, no retrying, not centralized, you can't customize success or failure emails, almost impossible to get non-engineers to use, etc.

Enter [Rundeck](http://rundeck.org/). It's a powerful system, though we're only using it in "glorified crontab with a web UI" mode, which is a fine way to start. We've now switched most of our cron jobs over to it, and we're working on moving our Windows Task Scheduler jobs as well.

![Rundeck logo](/public/images/rundeck-logo.png)

Below are some of the benefits for us.


## UI with single sign-on support

Rundeck has a web UI that supports single sign-on, so anyone on our team has automatic access to it. Among other things, this means people on other teams can write ETL scripts and set them up in Rundeck without involving engineering.

The UI is also good for engineering. It allows you to create jobs, run a job, view history, and check job output. Previously to look something up we'd have to hunt through old emails, or SSH or remote into a machine and scan through log directories. Big plus.

![Rundeck jobs UI](/public/images/rundeck-ui.png)

The UI is definitely better than Windows Task Scheduler (not hard to beat), but it's not a stellar example of design. It has a very [made by developers](http://blog.codinghorror.com/this-is-what-happens-when-you-let-developers-create-ui/) feel, with tons of detail upfront, and some common operations like looking at recent errors or log output hidden behind several clicks.


## History and output capturing

Rundeck logs all job activity, making it easy to debug when something went wrong and why. You can filter the activity history by task name, user, success/failure status, or time.

Once you've found the relevant run, you can drill down into it and look at all stdout and stderr output from a given command. For example:

![Rundeck error output](/public/images/rundeck-error-output.png)


## Success and failure emails

Windows Task Scheduler and crontab have very limited support for this, but with Rundeck it's easily configurable and simple to set up. You can tell it to send stdout/stderr output from the job as an attachment to the email.

Some of our scripts we just want to email on failure, others we want to email on success as well, to keep folks in the loop:

![Rundeck email notifications](/public/images/rundeck-email-notifications.png)

You can customize the subject line using job variables, and you can specify a custom Markdown email template if you want (we just use the default -- it's not pretty, but it is functional).


## Simple and advanced scheduling

Rundeck has a simple scheduling interface that allows you to run a job at a certain time every day or on selected days of the week. It also has a more advanced [crontab scheduling option](http://www.quartz-scheduler.org/documentation/quartz-1.x/tutorials/crontrigger) to allow you to run tasks on schedules like "every 15 minutes" or "1:30am every last Friday of the month".

![Rundeck simple scheduling](/public/images/rundeck-simple-scheduling.png)


## SSH support

Rundeck can run jobs locally on the machine Rundeck is installed on, or it can run jobs on remote machines [using SSH](http://rundeck.org/docs/plugins-user-guide/ssh-plugins.html). For most of our longer-running jobs, we have a single "worker server" that we run jobs on via SSH.

Rundeck has good support for multiple "node sources". For example, in our installation, our techops team has set up Rundeck to use our master node list from [Chef](https://www.chef.io/chef/).


## Ad-hoc commands

Rundeck also has a screen where you can run any [ad-hoc command](http://rundeck.org/docs/manual/commands.html) against one or more nodes. For example, you might want to check disk usage on a bunch of nodes:

![Rundeck ad-hoc commands](/public/images/rundeck-ad-hoc-command.png)


## Source control integration

One of the problems with UIs (like Task Scheduler) is that change tracking is hard or impossible. We wanted to be able to see who did what, and when. As of version 2.6.0, Rundeck has built-in support for source control integration via git. So we hooked it up to our git repo and now have a changed-tracked log of who did what to the job config.

All the project and job configuration is saved in YAML format, so is fairly easy to read, for example:

```
- description: ''
  executionEnabled: true
  group: Scheduled
  name: Main nightly scripts
  notification:
    onfailure:
      email:
        attachLog: true
        recipients: errors@oyster.com
  sequence:
    commands:
    - jobref:
        group: Task
        name: Dump live database to snapshot
    - jobref:
        group: Task
        name: Backup snapshot database
...
```


## Other features

Rundeck has plenty of other features, many of which we're not using yet:

* [Node matching](http://rundeck.org/docs/manual/nodes.html) and running jobs against multiple nodes
* Configurable [job options and multi-step workflows](http://rundeck.org/docs/manual/jobs.html)
* Other [notifications hooks](http://rundeck.org/docs/developer/notification-plugin.html): web hooks, HipChat, custom plugins
* [Open source](https://github.com/rundeck/rundeck) on GitHub, and very good [documentation](http://rundeck.org/docs/index.html)

Shout-out to our techops team who set up the Rundeck install for us and helped with various operational aspects.


## We're hiring!

If you like good engineering and automation and want to come work for us, [apply here](http://www.tripadvisor.com/careers/search-jobs?job_category=6&location=19&keywords=)!

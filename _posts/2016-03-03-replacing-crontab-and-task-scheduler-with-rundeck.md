---
title: Replacing crontab and Task Scheduler with Rundeck
author: Ben
layout: post
permalink: /replacing-crontab-and-task-scheduler-with-rundeck/
published: false
---

crontab on Linux and Task Scheduler on Windows are fine tools as far as they go. But they're very simple: difficult to schedule, no retrying, not centralized, you can't customize success or failure emails, almost impossible to get non-engineers to use, etc.

![Rundeck logo](/public/images/rundeck-logo.png)

Enter [Rundeck](http://rundeck.org/). It's a powerful system, though we're only using it in "glorified crontab with a web UI" mode, which is a fine way to start. We've now switched most of our crontab jobs over to it, and we're working on moving our Windows Task Scheduler jobs as well. Here are some of the benefits for us:

## UI with single-sign-on support

It has a decent UI that supports single sign on, so anyone on our team has automic access to it. This means our analytics team can write ETL scripts and set them up in Rundeck without involving engineering, saving everyone's time.

The UI is also good for engineering. It allows you to create jobs, run a job, view history, and check job output. Previously to look something up we'd have to hunt through old emails, or SSH/Remote Desktop into a machine and scan through log directories. Big plus.

![Rundeck jobs UI](/public/images/rundeck-ui.png)

The UI is definitely better than Windows Task Scheduler (not hard to beat), but I use the term *decent* advisedly. It has a very [made by developers](http://blog.codinghorror.com/this-is-what-happens-when-you-let-developers-create-ui/) feel, with tons of detail upfront, and some common operations like look at recent errors or log output hidden behind several clicks.

## Success and failure emails

Todo

## SSH support

Todo

## Saves job config to source control

One of the problems with UIs (like Task Scheduler) is that change tracking is hard or impossible. We wanted to be able to see who did what, and when. As of version 2.6.0, Rundeck has built-in support for source control integration via git. So we hoooked it up to our git repo and now have a changed-tracked log of who did what to the job config.

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
  schedule:
    month: '*'
    time:
      hour: '01'
      minute: '30'
      seconds: '0'
    weekday:
      day: '*'
    year: '*'
  scheduleEnabled: true
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

Rundeck has plenty of other features we're not using yet, such as:

* Node lists

Kudos to our techops team who set up Rundeck for us and helped with various operational aspects.

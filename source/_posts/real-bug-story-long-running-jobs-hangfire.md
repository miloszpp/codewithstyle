---
title: 'Real bug story: very long-running jobs in Hangfire'
url: 472.html
id: 472
categories:
  - .NET
  - Other topics
date: 2017-10-09 17:00:59
tags:
---

[Hangfire](https://www.hangfire.io/) is a great tool which can help you with doing background processing in .NET web applications. It's great for tasks such as background import or asynchronous processing of some events or requests. What's amazing about Hangfire is that it works very well in a setup where you have multiple instances of your application deployed behind a load balancer. In such case Hangfire can synchronize using database or Redis.

### Summary

You need to be extra careful when using long running jobs in connection with DisableConcurrentExecution  attribute.

### Problem

Some time ago I run into an interesting problem at work. I was using Hangfire to process requests from a queue. Users could add requests to the queue and than a Hangfire job would run every 5 minutes, take them off one by one and execute them. Processing of a single request was quite lengthy - it took about 2 minutes. The way I implemented it was to load all pending requests and execute them in a single run of the job. What's more, I wanted the requests to be processed sequentially. I applied the DisableConcurrentExecution  attribute in order to make sure that there is only a single instance of the job running at given time. The problem materialized itself when I added several hundreds requests to the queue. After some time the job started throwing the following error:

    Timeout expired. The timeout period elapsed prior to obtaining a connection from the pool

What happened was the following:

1.  The first job run noticed 200 new requests and started processing them. It would take ~400 minutes to finish the processing.
2.  Next executions of the job should not happen because of the DisableConcurrentExecution attribute. However, Hangfire does actually create the job instance - it's not starting it until the other execution is not finished.
3.  Our plumbing around Hangfire made it start a database transaction whenever a Job object is resolved from the IoC container. The database connection was obviously pulled from the connection pool.
4.  After some time we've had 100 job instances waiting for the first one to finish. Each one has borrowed a database connection from the connection pool.
5.  In consequence, at some point we were running out of the connections in the pool and the job started to crash.

![](/images/2017/10/Hangfire_–_Background_jobs_and_workers_for__NET_and__NET_Core.png)

### Solution(s)

It was a nasty issue and took some time to figure out. We've ended up with a workaround and increased the connection pool size because we knew that this huge batch of requests was a one-off thing. However, the whole design turned out to be flawed. It would be a much better idea to have more fine-grained jobs and process a single request in a single job execution. And this it my key takeaway from this bug story.
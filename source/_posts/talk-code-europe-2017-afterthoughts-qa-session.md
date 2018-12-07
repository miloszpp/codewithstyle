---
title: 'My talk at Code Europe 2017: afterthoughts and the Q&A session'
url: 275.html
id: 275
categories:
  - Misc
  - Other topics
  - Thoughts
date: 2017-05-27 15:58:14
tags:
---

Two days ago I delivered my first conference tech talk at [Code Europe](http://codeeurope.pl/). It's a major programming conference taking place in three different cities in Poland (I presented in Warsaw). My talk was based on one of the posts I wrote for this blog: [Building serverless web application with Angular 2, Webtask and Firebase](http://codewithstyle.info/building-serverless-web-application-angular-2-webtask-firebase/). ![](http://codewithstyle.info/wp-content/uploads/2017/05/584151c052d4c3.81439466_625.jpg) It was a great experience. I've spend a lot of time practicing, trying to implement all the knowledge I've learned from various blog posts about public speaking and tech talks (I really recommend [this](https://www.sqlskills.com/blogs/paul/public-speaking-a-primer/) and [this](https://www.hanselman.com/blog/11TopTipsForASuccessfulTechnicalPresentation.aspx)). I had a lot of support from my [contracting agency](https://www.7n.com/) for which I'm very grateful. It was a lot of hard work but it's totally worth it. The way you feel after the presentation when people come over to thank you for a great talk is the best reward you can get. Besides, I feel like the whole process made me more confident and a better communicator in general. ![](http://codewithstyle.info/wp-content/uploads/2017/05/code_europe_milosz_piechocki-1024x768.jpg) My talk included a demo in which I built a small web application. This application allowed the audience to ask questions in real-time (using their phones) which I answered at the end of the talk. This turned out to be a funny thing to do since people were posting anonymously and some of the questions were not related to the talk at all :). However, I've decided to pick some and post here along with some more elaborate answers. **What are the disadvantages sides of serverless?** I only mentioned vendor lock-in (the fact that it's difficult to move your stuff between different service providers; this is particularly true with BaaS) while in fact there are more points to consider. [This article](https://www.martinfowler.com/articles/serverless.html) does a great job enumerating them. In short:

*   Vendor control - since you are outsourcing your infrastructure, you no longer have control over it; if something breaks the only thing you can do is to hope that somebody on the other side will fix it quick
*   Multitenancy - in FaaS your code shares hardware and infrastructure with someone else's code; this can lead to problems such as one user using up all the available resources; obviously service providers are trying hard not to allow such a situation
*   Repetition of logic across various platforms - one of the guys asked explicitly about that; if you have clients developed in multiple platforms (e.g. JavaScript and .NET) than you will end up with duplicated logic; see below

**How do you authenticate users so only data they have permission to is pulled to their instance of the client?** In Firebase Authentication is tightly integrated with Real-time database. This means that you can use a special variable called auth  when creating validation rules. Let's say we store contacts in a database. For each user id we store a list of contacts.

{
  contacts: {
    "user1": \[{ ... }, { ... }\],
    "user2": \[{ ... }, { ... }\],
    ...
  }
}

With the below rule we can define that an authenticated user only has access to the data under his user id.

"rules": {  
    "contacts": {
      "$userId": {
        ".write": "auth.uid == $userId",
        ".read": "auth.uid == $userId",
     }
}

**Won't such automation push the "regular programmers" out of the market at some point?**

I think that serverless might remove some of the sysops jobs. It's effectively a form of outsourcing and companies usually outsource stuff so that they don't need to hire people full-time.

From the developer's perspective, I think the best thing we can do is to stay up to date and learn serverless.

**Can I do server processing doing some tasks without client-side?**

The author clarified that what he meant is whether it's possible to execute serverless code in other ways than by making a web request from some client. It totally is, for example Webtask.io has a scheduled execution mode in which you can define that your task should be triggered on some sort of schedule.

Cloud Functions in Firebase have an even longer list of possible triggers. For example, you can setup your function to execute when something specific happens in your Realtime Database or when some condition related to Google Analytics is fulfilled.

**How about access from no browser client in context off logic duplication**

That's a totally valid point. If you have clients developed in multiple platforms (e.g. JavaScript and .NET) than you will end up with duplicated logic.

On the other hand, serverless might help you avoid duplication in some other places. For example, you can have your codebase shared between the client and your FaaS functions. During the demonstration in my talk I used exactly that - I reused the config file and the domain model in the Webtask.
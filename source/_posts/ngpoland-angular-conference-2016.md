---
title: ngPoland - Angular Conference 2016
tags:
  - angular
  - conferences
url: 103.html
id: 103
categories:
  - Angular
  - Articles
  - Web
date: 2016-11-22 19:49:27
---

Today I attendend [ngPoland](http://ng-poland.pl) \- the first international conference devoted to Angular in Central Europe. I've had some really good time there and decided to share some of the amazing things I learned about. First of all, I was surprised to learn how good some of the speakers were at catching people attention and making sure that everyone stays awake. The conference was pretty intense (I counted 15 talks) so it was quite a challange. It was inspiring to see how got can one become at public speaking and working with large audiences. 

![Photo by Phil Nash from Twilio](/images/2016/11/ngpol-1024x366.jpg)

The key takeaway for me is to deffinietly **look into [Redux](http://redux.js.org)** (the presentation by [Nir Kaufman](https://twitter.com/nirkaufman)). The framework introduces a great idea from functional programming to the frontend world. Redux allows you to express your application's logic as a set of reducer functions which are applied on the global, immutable state object in order to produce the "next version" of state. Thanks to that, it's much easier to control and predict state transitions. Similiarity to the [State Monad](https://wiki.haskell.org/State_Monad) seem obvious. 

Another very interesting point was the presentation by [Acaisoft's](http://acaisoft.com) founder who showed a live demo of an on-line quiz app with real-time statistics. The application was implemented in Angular 2 with **serverless architecture** ([AWS Internet of Things](https://aws.amazon.com/iot/) on the backend side), **event-sourcing** and **WebSockets**. It was exciting to observe a live chart presenting aggregated answers of 250 conference participants who connected with their mobiles. 

Definietly the most spectacular talk was the one about **using Angular to control hardware connected to a Raspberry Pi device** (by [Uri Shaked](https://twitter.com/UriShaked))! The guy built a physical [Simon game](https://en.wikipedia.org/wiki/Simon_(game)) that was controlled by an Angular app. Thanks to [angular-iot](https://github.com/urish/angular2-iot) he was able to bind LED lights to his model class. The idea sounds crazy but it's a really convincing demonstration that Angular can be useful outside of the browser. If you are interested, you can read more [here](https://medium.com/@urish/building-simon-with-angular2-iot-part-2-ee3a270747b5). 

Last but not the last, I have to mention the workshop about **TypeScript 2** (again by [Uri](https://twitter.com/UriShaked)) which I attended the day before. Although I knew TypeScript before, it was interesting to learn about the new features such as **null strictness** and **async/await**. Coming from a C# background, it's very easy to spot the Microsoft touch in TypeScript. I believe the language is evolving into the right direction and I'm happy to see more and more ideas from functional programming being incorporated in other areas. 

Wrapping up, I think the conference was very convincing at demonstrating how much stuff is happening around frontend development. I like the general direction towords each its evolving and I hope that I will have many opportunities to work with all the new stuff.
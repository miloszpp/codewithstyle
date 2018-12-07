---
title: NgPoland 2017 conference summary
url: 540.html
id: 540
categories:
  - Angular
  - Thoughts
  - Web
date: 2017-11-25 10:59:27
tags:
---

Last Tuesday I gave a talk about using **Functional Programming in Angular** at the [NgPoland](http://ng-poland.pl) conference - the biggest Angular conference in Central and Eastern Europe (and big it was - over 600 Angularians in one place). While I'm planning to write an article based on my talk, first I would like to post some notes from the other talks I saw. ![](https://codewithstyle.info/wp-content/uploads/2017/11/23736340_10210922780092493_4316555156399740919_o.jpg) I will risk a statement that the choice of topics in this conference proves that **frontend programming is getting more and more functional**. At least four talks had something to do with functional programming!

*   **Understand NGRX by building a Store** \- NGRX is pretty functional itself as it embraces immutability and pure functions
*   **High-Performance Applications with Angular** \- many ways of improving performance in Angular are based on functional concepts - memoization, pure pipes, `OnPush` change detection strategy (hence immutability)
*   **How to use Functional Programming with Angular and why it’s a good idea** \- quite obviously ;)
*   **Understanding High Order Observables** \- higher-order functions and function composition

Since I was a speaker I didn't have a chance to see all talks, so **the list is not be complete**.

### Keynote - Understand NGRX by building a Store by Todd Motto

While it might seem that the talk was about using NGRX, it was actually more about how to build your own NGRX! NGRX is a Redux implementation with a reactive API which plays well with the rest of Angular. The talk included a large live coding session where Todd explained the Redux design pattern by building an application from scratch in vanilla JavaScript. I think it's a great idea, especially given that we're surrounded by so many frameworks and sometimes we use them without having understood how they work inside.

### Angular - Learning redesigned by Dariusz Kalbarczyk

This short talk was given by the organiser of the conference. While it was not technical, I think it was really inspiring. Dariusz explained how to benefit the most from the conference by setting up clear learning goals.

### High-Performance Applications with Angular - Nir Kaufman

A truly amazing talk, packed with tips and suggestions on how to tackle performance issues in Angular applications. First, Nir stated that optimizing Angular performance is mostly about shortening the change detection process. Next, he talked about many ways to achieve this including: using the `OnPush` change detection strategy, manually controlling change detection by detaching `ChangeDetectorRef`, using memoization for caching results of pure functions, using pure pipes, running code outside of `NgZone`, and many more.

### Login and Access Control in Angular - Manfred Steyer

Very insightful and well delivered talk about architecting authentication in SPA applications. Manfred presented many different options including using the OAuth2 standard and 3-rd party providers such as Firebase. He has also talked about popular security attacks and how to tackle them.

### How to use Functional Programming with Angular and why it’s a good idea - Miłosz Piechocki

As mentioned, I will write a separate article with the summary of my talk.

### Building Angular2 & Firebase application - Fáris Ismail

The talk was delivered by an early adopter of Cloud Firestore - the new storage offering from Firebase - so we've been given some first-hand experiences. Since I was stuck with the Realtime Database, the presentation was perfect to me - it explained all of the advantages of the new Firestore solution. The live demo which involved hundreds of people interacting with the application real-time (and displayed on a huge cinema screen) was pretty impressive too!

> [#ngPolandConf](https://twitter.com/hashtag/ngPolandConf?src=hash&ref_src=twsrc%5Etfw)  
> This guy currently collect more emails than recruiters :D :D [pic.twitter.com/CTyeD1nQ6G](https://t.co/CTyeD1nQ6G)
> 
> — Frantisek Ferko (@FrantisekFerko) [November 21, 2017](https://twitter.com/FrantisekFerko/status/933001648313720832?ref_src=twsrc%5Etfw)

### Angular 5 has arrived - Bartłomiej Narożnik

Nice and succinct summary of the new features introduced in Angular 5. Bartłomiej tried to answer the question whether the new version is mature enough to upgrade. Additionally, he presented some cool statistics about the Angular repository. ![](https://codewithstyle.info/wp-content/uploads/2017/11/DPJmWT-XUAEhhmM-1024x768.jpg)

### Understanding High Order Observables - Gerard Sans

Gerard talked about how to use the [RxJS Marbles](http://rxmarbles.com/) testing library to visualize and understand different RxJS operators. It allows you to describe test prerequisites and expectations using... ASCII art :) The library is actually used for testing the RxJS library - you can find some tests on GitHub. Great for experimenting and wrapping your head around hundreds of RxJS operators.
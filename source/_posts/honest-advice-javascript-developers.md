---
title: Honest advice for JavaScript developers
url: 422.html
id: 422
categories:
  - Other topics
  - Thoughts
  - Web
date: 2017-08-31 16:23:17
tags:
---

### Modern JavaScript is considered hard

Have you noticed the abundance of tutorials, introductions and guides devoted to post-ES6 JavaScript? All these conferences, workshops, bootcamps where you get a chance to learn about _async/await_ or _Redux_? The books and on-line courses? I think it shows how many people are struggling to grasp all these new JavaScript topics. The point I'd like to make is that **all of these new ideas in JavaScript are not new in terms of Computer Science**. Therefore, you can really get ahead of others and future-proof your career by investing time into learning about existing, (superficially) less sexy, programming languages and frameworks. \[caption id="attachment_430" align="aligncenter" width="700"\]![](https://codewithstyle.info/wp-content/uploads/2017/08/rxjs-1024x683.jpg) Source: https://imgflip.com/i/1ar013\[/caption\]

### Revolution in JavaScript

If you've been writing JavaScript for some time you must have noticed the revolution that is happening for some time. The dawn of ES6 (a.k.a. ECMAScript 2015) made JavaScript a much more complex language. Just look at the list of new language features at [es6-features.org](http://es6-features.org/) \- it's quite impressive given that you're looking at a new version of existing language and not a completely new language. What's more, you have to make more and more effort just to keep up with the language - [ECMAScript standard will now be updated every year](https://thenewstack.io/whats-new-es2016/). Although the 2016's and 2017's versions have not been as revolutionary as the 2015's, the concepts introduced in them are definitely non-trivial. The explosion of new JavaScript frameworks doesn't help. You can find all the new concepts introduced in Redux or RxJS intimidating even if you are a seasoned JavaScript developer.  

### The new-old ideas in JavaScript

There are far too many seemingly innovative concepts in JavaScript to mention all of them here. Instead, let me focus on a few examples just to give you the general idea.

#### Promises

Promises in JavaScript were introduced as a cure to the so-called callback hell. Asynchrony in JavaScript is unavoidable. Before promises the only way to handle asynchronous operations was to pass callbacks as arguments to asynchronous functions (such as loading some data from the backend). Multiple asynchronous operations often mean that you must have multiple levels of nesting of callbacks which makes the code less readable and difficult to maintain. Promises provide abstraction over a computation that might not have finished yet. Having such an abstraction lets you elegantly chain and compose async operations. This [amazing animation](https://async-await.xyz/) illustrates the concept (along with async/await): ![](https://codewithstyle.info/wp-content/uploads/2017/08/js-callbacks-promises-asyncawait-300x225.gif) The idea of promises is not new at all. In fact, a very similar concept (Tasks) has been present in C# since .NET Framework 4.0 (year 2010). Java had Futures long before that but they weren't composable.

task
  .ContinueWith(t => GetMoreData()).Unwrap()
  .ContinueWith(t => GetMoreData2(t.Result)).Unwrap()
  .ContinueWith(t => t.Result.ToLower());

Above you can see some C# code which uses Tasks (the equivalent of Promises). You have to admit that there are similarities with the JavaScript version. Rest assured that having understood the idea behind Tasks, wrapping your head around Promises is trivial.

#### Reactive programming

Reactive programming has been introduced into JavaScript in the form of the [RxJS](http://reactivex.io/) library. It became quite popular, even [making its way into the Angular framework](https://medium.com/google-developer-experts/angular-introduction-to-reactive-extensions-rxjs-a86a7430a61f). \[caption id="attachment_426" align="aligncenter" width="700"\]![](https://codewithstyle.info/wp-content/uploads/2017/08/reactive-stream-example-1024x482.png) Illustration of event stream processing. Source: http://reactivex.io/documentation/observable.html\[/caption\] RxJS introduces the concept of Observables. It's another way to handle asynchrony. Instead of setting up callbacks that should fire when some event happens you deal with a **stream of events**. It's very powerful because you can perform different operations on that stream of events - for example filter out some events or join two event streams. En example of event is someone clicking something in your application. Reactive programming builds on the Observable pattern which has been with us since the dawn of Object Oriented Programming. I could find [references to Rx.NET](http://www.hanselman.com/blog/HanselminutesPodcast198ReactiveExtensionsForNETRxWithErikMeijer.aspx) (reactive extensions library for .NET) as early as from 2010. Besides, reactive operators (such as map, filter or reduce) are derived directly from functional programming and languages as old as Haskel (1985). Below you can find a comparison between reactive code in JavaScript and .NET. Again, there are some minor differences but the concept is identical.

observable.Select(n => n * n)
          .Where(n => n % 2 == 1)
          .Zip(observable.Skip(1), (x, y) => x * y);

observable
    .map(n => n * n)
    .filter(n => n % 2 == 1)
        .zip(observable.skip(1), (x, y) => x * y);

#### Async/await and generators

The async/await syntax is one more way to deal with asynchrony. It lets you write asynchronous code which looks as if it was synchronous! Here is an example from the Mozilla docs:

async function getProcessedData(url) {
  let v;
  try {
    v = await downloadData(url); 
  } catch(e) {
    v = await downloadFallbackData(url);
  }
  return processDataInWorker(v);
}

Async/await became part of JavaScript only recently - as part of the ES2017 standard. It has been present in C# since version 5 of the language (year 2012). Besides, async/await is strongly related to the concept of generators (another JavaScript novelty introduced in ES6). Generators were present in Python since version 2.2 (year 2001).

#### Event sourcing

Finally, an example not related to asynchronous programming. Redux is a library for managing complex state. Instead of representing your state as a mutable object(s) which changes whenever the user interacts or some data comes from the backend, you can look at it as a stream of immutable objects. Each consecutive object represents the new version of the state. New versions are produced by a **reducer function** which takes the previous version and an **event** and computes how the event affects the new shape of the state. This idea borrows heavily from architectural patterns such as event sourcing and CQRS (as [admitted](http://redux.js.org/docs/introduction/Motivation.html) by Redux creators). This [article from 2005](https://martinfowler.com/eaaDev/EventSourcing.html) explains event sourcing in detail. I can't point to any particular language here but chances are that if you have done any backend programming you might have heard about event sourcing long before it made its way into the JavaScript world.

### Summary

These are just a few examples of seemingly new ideas in JavaScript which have actually been around for some time. I make a lot of references to .NET and C# because it is the language I know best but in fact C# is pretty progressive in terms of introducing new language features. I hope you feel convinced that spending some time to broaden your horizons and learn some backend language is totally worth it. If not, let me know your thoughts in the comments section! If you liked the article please consider sharing it.
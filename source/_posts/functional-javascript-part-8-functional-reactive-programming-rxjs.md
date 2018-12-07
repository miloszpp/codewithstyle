---
title: 'Functional JavaScript part 8: functional-reactive programming with RxJS'
url: 441.html
id: 441
categories:
  - JavaScript
  - Web
date: 2017-09-24 15:47:53
tags:
---

Let's talk about RxJS - another concept that has recently established itself as an important part of modern JavaScript. In JavaScript applications you sometimes have to deal with streams of asynchronous events. For example, events could be produced by the user (button clicks, key presses) or pushed by a backend server (via web sockets or some other mechanism). ![](https://codewithstyle.info/wp-content/uploads/2017/09/rxjs-300x300.png) Given that applications are becoming more and more complex, it might become tricky to manage these streams in a traditional way (with callbacks). **Reactive programming** is a programming paradigm in which streams of data are central and therefore it's much easier to work with them. [**RxJS**](https://github.com/Reactive-Extensions/RxJS) is a **functional reactive programming** library. It means that it leverages functional techniques to facilitate dealing with event streams. In simple words, it lets you use the same [operations that you learned to perform on arrays](https://codewithstyle.info/functional-javascript-part-2-array-operations/) on event streams. **This post is part of the [Functional Programming in JavaScript series](https://codewithstyle.info/functional-programming-javascript-plain-words/).**

### Creating observables from events

RxJS introduces the concept of **o****bservable**. An observable represents a stream of data (or events). Given an observable you can **subscribe** to it. When subscribing you provide a callback which will be fired every time a new item is popped out of the stream. Let's take a text input field and create an observable that will represent the stream of keys typed into it.

<script src="https://unpkg.com/@reactivex/rxjs@5.0.0-beta.12/dist/global/Rx.js"></script>
<input id="textInput" type="text" />

var textInput = document.getElementById("textInput");
var keyStream = Rx.Observable.fromEvent(textInput, 'keyup');

As you can see, it's pretty straightforward. RxJS allows us to easily convert classic JavaScript events to observables. Let's now subscribe to this observable.

keyStream.subscribe(e => console.log("Key pressed: ", e.key));

### Introducing functional operators

I promised that RxJS has something to do with functional programming. Do you remember the filter operation? It would take an array and a predicate function and filter out elements which don't satisfy the predicate. With RxJS you can treat the stream of events as if it were a regular JavaScript array. Can you guess the result of applying filter to our keyStream?

keyStream
  .filter(e => e.key === e.key.toUpperCase())
  .subscribe(e => console.log("Capital key pressed: ", e.key));

Calling filter  on an observable creates a new observable which will only emit events that satisfy the predicate. In the above example we're only passing on key presses if they are capital letters. ![](https://codewithstyle.info/wp-content/uploads/2017/09/filter.png "filter") Other array operations that you've learned such as map  or reduce  can also be applied to observables.

### Combining observables

Let's say that every time a user types an upper case character we would like to perform a search using some REST API. We can do this elegantly using RxJS! We need to start thinking in terms of streams. We have a stream of key presses. Let's transform it into a stream of search results coming from the REST service. What would we use if we wanted to transform an array-of-something to an array-of-something-else? We would use map. And this is exactly what we will use now.

keyStream
  .filter(e => e.key === e.key.toUpperCase())
  .map(e => fetch("https://jsonplaceholder.typicode.com/posts?search=" + e.key));

We use fetch to make the call to the backend server. However, fetch returns a promise so what we've done so far is transformed the stream of keys to a stream of promises. Not exactly what we wanted. Fortunately, it's trivial to convert a promise to an observable.

keyStream
  .filter(e => e.key === e.key.toUpperCase())
  .map(e => Rx.Observable.fromPromise(fetch("https://jsonplaceholder.typicode.com/posts?search=" + e.key)));

There is one more problem with this though. At the moment we're mapping each key press to an observable. Therefore, we've transformed an observable of keys to an observable of observables! In other words, for each key we will get a separate observable. It's not perfect. We would much rather interact with a single observable instead. For that we need to combine the observables from the stream into a single one. Fortunately, there is an operator for that!

keyStream
  .filter(e => e.key === e.key.toUpperCase())
  .flatMap(e => Rx.Observable.fromPromise(fetch("https://jsonplaceholder.typicode.com/posts?search=" + e.key)))
  .subscribe(data => console.log(data));

flatMap  takes a stream and a function mapping each event to another observable. Then it combines all of the resulting observables into a single one. ![](https://codewithstyle.info/wp-content/uploads/2017/09/flatMap.png "flatMap") flatMap  is an extremely important operation in the world of functional programming. It is a very high-level abstraction which allows you to combine structures. For example, there is a variant of flatMap  for array operations in the [lodash](https://lodash.com/docs/4.17.4#flatMap) library. Its purpose is to take an array of arrays and combine the nested arrays into a single array. Can you see the pattern emerging?

### Other non-functional operators

There are many other interesting operators in RxJS which are not rooted in functional programming but are definitely worth exploring. Let's finish off our example with one more improvement. At some point we might realize that calling the REST service after every key press is killing performance in our app. One possible solution to this is to only fire the request after some time passed since the last request. This would be a nightmare to implement with callbacks or promises. With RxJS it's a matter of a single line

keyStream
  .filter(e => e.key === e.key.toUpperCase())
  .debounceTime(500)
  .map(e => Rx.Observable.fromPromise(fetch("https://jsonplaceholder.typicode.com/posts?search=" + e.key)));
  .subscribe(data => console.log(data));

debounceTime  will do exactly what we want. It will wait 500 milliseconds after each key press. It will only emit this key press if there are no successful key presses in the incoming 500 milliseconds.

### Summary

I've just scratched the surface of RxJS in this post. There are many other interesting operators which I encourage you to explore. Most importantly, it should all be much easier now once you are familiar with array operations. Once again you can see how universal the ideas in functional programming are. If you have any issues understanding anything in this post or if you simply would like to provide feedback, please leave a comment below. I want this course to be as good as possible and I need your help for that! If you found this post helpful, please consider sharing it on Facebook or Twitter.
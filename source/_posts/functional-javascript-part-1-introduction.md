---
title: 'Functional JavaScript part 1: introduction'
url: 307.html
id: 307
categories:
  - JavaScript
  - Web
date: 2017-07-29 09:36:31
tags:
---

Welcome to the first post of the series about functional JavaScript! In this series of articles I'll show you some basic functional programming techniques applied to JavaScript as well as explain how functional concepts are used in modern JavaScript frameworks. **This post is part of the [Functional Programming in JavaScript series](https://codewithstyle.info/functional-programming-javascript-plain-words/).**

### Why learn functional programming?

![](http://codewithstyle.info/wp-content/uploads/2017/07/javascript-300x300.png) Recently, we've observed a massive trend of embracing functional programming. Many object oriented languages started to incorporate functional programming features (e.g. streams in Java, pattern matching in C#, etc.). In the front-end world there is an abundance of examples of how functional programming concepts made its way into popular frameworks:

*   unidirectional data flow in React and Angular
*   immutability, event sourcing, higher-order functions in Redux
*   reactive operators in RxJS
*   functional-reactive frameworks such as Cycle.js

In my experience, the above topics are difficult to grasp for people who are not familiar with functional programming at all. Learning the functional ways gives you a great head-start when dealing with these concepts. And given that the trend is definitely in favor of FP, we may anticipate even more FP concepts coming into the JavaScript world. Obviously, there are more benefits to FP. Using functional techniques usually results in cleaner, less buggy, self-descriptive code. Besides, it is really fun and intellectually rewarding. **Whether you like it or not, you are probably going to use functional programming to some extent.** Why not take initiative and learn it properly?

### What is functional programming?

Many people associate programming with writing down some **instructions** telling the computer what to do. This is just one of many approaches to programming. We call this approach **imperative programming**. In imperative programming, your primary building block are **statements**. There are many kinds of statements - variable assignments, if  statements, for  loops and more. Your program executes from top to bottom, taking the statements one by one and running them. Functional programming is different. In functional programming, instead of specifying exactly what to do by providing a list of instructions, you define your program as a **function** which takes some input and produces some output. But how can a complex program fit into a function? The answer lies in composition. You break down your program into multiple smaller functions and compose them. As a result, when writing functional code, you don't need to be as specific as while writing imperative code. Because of that, the code is more readable, less prone to bugs and mistakes and more self-describing. The below example illustrates what I mean by that. In the imperative way, you need to be very specific about how to use an index variable to iterate over an array. You need to provide details such as how to initialize the variable and how to increment it. There are many places where you can get it wrong and the intent is not that clear by looking at the code. Compare it with the functional piece. We abstract away the details of how to iterate over an array. The code is very readable and catches the intent of the programmer.

var books = \[ "Gone with the Wind", "War and Peace" \];

// imperative way
for (var i = 0; i < books.length; i++) {
  console.log(books\[i\]);
}

// functional way
books.forEach(book => console.log(book));

### Is JavaScript functional?

Although JavaScript is far from being a pure functional language, it has some support for functional programming and makes writing functional code pretty natural.

#### Passing functions

Most importantly, functions are first category citizens in JavaScript. It means that you can assign functions to variables and do with them anything you would do with any variable. Most importantly, you can pass functions as parameter to another function or return it from a function. Thanks to that, JavaScript makes it possible to naturally compose functions.

function updateHeader(updater) {
  var headerEl = document.getElementById("header");
  updater(headerEl);
}

function setColorToRed(element) {
  element.setAttribute("style", "color:red");
}

function capitalizeContent(element) {
  element.innerText = element.innerText.toUpperCase();
}

updateHeader(Math.random() > 0.5 ? setColorToRed : capitalizeContent);

In the above example, we can see that JavaScript allows us to **treat functions as data**. We pick one of the functions at random and pass it as an argument to another function.

#### Anonymous functions

Another aspect of JavaScript which makes it FP friendly is the support for anonymous functions. In functional programming we create functions all the time. Therefore, it wouldn't be very convenient if every function had to be named. Fortunately, in JavaScript we can create functions without naming it.

updateHeader(function (element) {
  element.innerText = element.innerText.toUpperCase();
});

Actually, thanks to the ES6 standard we can write anonymous functions in a more concise way:

updateHeader(element => {
  element.innerText = element.innerText.toUpperCase()
});

updateHeader(element => element.innerText = element.innerText.toUpperCase());

#### Clojures

One very useful aspect of anonymous functions is called **clojures**. Thanks to the clojure mechanism, it's possible to reference variables from outside of the scope of the function inside the anonymous function's body. This mechanism is especially useful when returning functions.

function getHeaderUpdater() {
  var headerEl = document.getElementById("header");
  return function() {
    headerEl.innerText = element.innerText.toUpperCase();
  };
}

By returning a function which references a variable from outer scope, we've created a clojure. It means that the returned function has captured the headerEl  variable - it will remain in memory even when the control flow exits getHeaderUpdater.

#### Immutability

As we will soon learn, immutability is an important aspect of FP. The idea of immutability is that you shouldn't mutate (modify) objects. Once you assign some properties to an object, they should stay as they are until the very end. If you need to change some property, you should return a new copy of the object instead of modifying it. Immutability makes your code much less prone to errors - you can make assumptions about your objects and you don't need to worry that they will be changed from a completely different place in your code. JavaScript doesn't support programming very well - it was designed as a language to **mutate** the DOM model. However, ES6 adds support for **destructuring** which can be helpful with regards to immutability. What's more, there are libraries which can help you enforce immutability such as [immutable.js](https://facebook.github.io/immutable-js/).

### Summary

I hope that I've convinced you that it's worth spending some time on learning functional programming. Since you already know JavaScript and at the same time the language is pretty well suited for writing functional code, there are really no excuses not to give it a try! Stay tuned and get ready for the [next article](http://codewithstyle.info/functional-javascript-part-2-array-operations/) in which we will talk about dealing with arrays the functional way. If you have any issues understanding anything in this post or if you simply would like to provide feedback, please leave a comment below. I want this course to be as good as possible and I need your help for that! If you found this post helpful, please consider sharing it on Facebook or Twitter.
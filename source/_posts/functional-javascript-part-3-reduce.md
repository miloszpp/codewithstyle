---
title: 'Functional JavaScript part 3: reduce'
url: 323.html
id: 323
categories:
  - JavaScript
  - Web
date: 2017-08-04 10:00:53
tags:
---

In the [previous post](http://codewithstyle.info/functional-javascript-part-2-array-operations/) we've looked at three very useful functions: forEach, map and filter. Let's now look at reduce  \- a bit more complicated function which shows the real power of functional programming. **This post is part of the [Functional Programming in JavaScript series](https://codewithstyle.info/functional-programming-javascript-plain-words/).**

### Reduce

I will describe to you a real-world problem which my colleague was dealing with at work. He was working on a filtering mechanism where the user could define conditions on a column in a table. The column would usually be mapped to a property of an object. However, it should also support nested properties. At some point in his code, he had to solve the following problem: _given a string representing the path of nested properties in an object, return the value of that nested property_. We can't assume anything about the length of the path - properties can be arbitrarily nested. For example, given path author.address.city.size  and object:

{
  author: {
    name: "John",
    address: {
      street: "Postępu",
      city: {
        name: "Warsaw",
        size: "large"
      }
    }
  }
}

his function should return "large". As usual, let's first approach the problem in a traditional, imperative way.

var path = "author.address.city.size";
var pathParts = path.split(".");

var current = book;
for (var i = 0; i < pathParts.length; i++) {
  var currentPart = pathParts\[i\];
  current = current\[currentPart\];
}

console.log(current);

Firstly, we split the path so that instead of a single string, we deal with an array where each element is a single property in consecutively nested object. Next, we initialize a helper variable current to the object which we wish to inspect (book  in our case). This variable will store the currently nested objects as we descend deeper and deeper inside the object. We iterate over the path parts and for each part we use it to go one level deeper. Once we are done, we end up with the desired value. And here comes the functional version:

pathParts.reduce((current, currentPart) => current\[currentPart\], book);

A single line! Isn't it awesome? Ok, I cheated a bit by omitting some code, but it's still one line versus four.

### But how does it work?

With previous higher-order functions (forEach , map  and filter) we passed a function to be applied on each element. This time it is a bit different. We pass a function which takes two arguments - an **accumulator** and the currently processed element. What is accumulator? It is a value which represents the intermediate result of processing. Reduce  looks at consecutive elements of the provided array. The accumulator should always hold a valid result for all elements processed so far. In other words, accumulator accumulates the results of processing consecutive array elements. It's easiest to understand when compared with the imperative code above. The current  variable is actually an accumulator. For every loop step, we take the current level on nesting (stored in the accumulator) and use it to get to the next level of nesting. Then, we store the result as the new accumulator. Reduce follows exactly the same process but it hides the details of looping and initializing the accumulator. Let's have a deeper look at how reduce  should be called. It takes two parameters:

*   **reducer function** which transforms the current accumulator value and an array's element to next accumulator - it encapsulates the loop step in the imperative code where we did exactly the same thing - processed the next array element and produced a new accumulator
*   **initial accumulator** \- because we need to start with some value of accumulator (as in the imperative code, where we initialize current = book)

### Another example

Let's have a look at another example in which we will use reduce  in a different way. Our use case is much simpler now: _take an array of numbers and return a sum of its elements_. Pause for a moment now and try to come up with a way to use reduce  to solve this problem. Let's try to figure out what should we store in the accumulator. I've already said that for every step the accumulator should store the valid result for the currently processed array elements. In our case the _result_ is the sum of currently processed elements - that's exactly what we will store in the accumulator. Next, let's see what the reducer function should do. As we know, it should take the current array element and the accumulator and produce the new accumulator. Since we agreed that the accumulator will store the sum of elements processed so far, then in order to produce a new accumulator we simply need to add the current array element to the old accumulator! ![](http://codewithstyle.info/wp-content/uploads/2017/08/reduce-explained.png "reduce explained") Finally, the initial accumulator value. Since we're going to add array elements to it, so it's best to initialize it simply to 0. Wrapping it up, this is how our reduce  call should look like:

var numbers = \[1, 2, 3, 4, 5\];
var sum = numbers.reduce((accumulator, currentNumber) => accumulator + currentNumber);
console.log(sum);

### Summary

In this article I've explained one of the most powerful concepts of functional programming - the reduce  function. Reduce has abundance of applications. [Redux](http://redux.js.org/docs/introduction/) is based exactly on this concept. In Redux, we write reducer functions which take the current state and an action and produce a new state. Does it sound familiar now? I will dedicate a separate episode of this course to Redux, so stay tuned. Another great example is the [MapReduce](https://en.wikipedia.org/wiki/MapReduce) programming model used in parallel processing of large data sets. I hope I've got you at least a little bit excited about reduce :-) I will conclude the part about array operations with [a post about lodash](https://codewithstyle.info/functional-javascript-part-4-lodash/) \- a library which extends the collection of higher order functions available in vanilla JavaScript with many more operations. If you have any issues understanding anything in this post or if you simply would like to provide feedback, please leave a comment below. I want this course to be as good as possible and I need your help for that! If you found this post helpful, please consider sharing it on Facebook or Twitter.
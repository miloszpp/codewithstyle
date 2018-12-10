---
title: 'Functional JavaScript part 2: array operations'
url: 317.html
id: 317
categories:
  - JavaScript
  - Web
date: 2017-08-02 13:48:27
tags:
  - javascript
  - functional programming
---

In the [first post of the series](http://codewithstyle.info/functional-javascript-part-1-introduction/), we've discussed which elements of the JavaScript language might be useful when writing functional code. Let's now see the most obvious example of functional programming in JavaScript - array operations. 

**This post is part of the [Functional Programming in JavaScript series](https://codewithstyle.info/functional-programming-javascript-plain-words/).**

### Imperative approach to arrays

Arrays are a basic programming construct present in most of programming languages. They let us deal with situations where we need to store or operate on multiple instances of some piece of data. As I already explained, imperative programming is all about executing instructions in a sequence. We can use various kinds of loops to deal with arrays. Usually, the loop body describes what to do to each element of an array. Let's have a look at the for loop in JavaScript. The below example iterates over an array and prints each element.

```javascript
var books = [ "Gone with the Wind", "War and Peace" ]; 
for (var i = 0; i < books.length; i++) {
  console.log(books[i]);
}
```

This code looks very familiar. Is there anything wrong with it? The problem that we are solving here can be stated as _print every element of given array of books_. However, there is much more going on in this piece of code:

*   Declare an index variable and initialize it to zero
*   Increment this variable by one until it exceeds the length of the array
*   For each step use that variable to access i-th element of the array and print it

Won't you agree that while our intent is very simple, the above process is quite complex? What's worse, there are many things that we can get wrong in this code - initialize the index to 1, use <= instead of <, forget about incrementing the index variable. This is because we have to be very specific about how to iterate the array and it's quite easy to miss something. We could rewrite this code using the while  loop. However, it wouldn't really buy as anything since the process would be roughly the same, only written in different way. _Actually, using the for...of construct from ES6 would be a big improvement in readability while still being imperative code. I'll talk about it in one of the future posts. For now, let's skip over it._

### ForEach

As I've already mentioned in the first post of the series, we can rewrite this code in a more functional way:

```javascript
books.forEach(function (book) {
  console.log(book);
});
```

What's going on here? We're calling the build-in forEach  method on the books  array. forEach is kind of special because it accepts a function as one of it's parameters. In functional programming, we have a special name for functions which accept (or return) other functions as parameters - **higher order functions**. But what does forEach  actually do? It runs given function on every element of an array. So basically, it does exactly the same thing as the for loop above but it hides the ugly details of having an index variable and incrementing it. Isn't that cool? Let's break it down a bit. Instead of using an anonymous function, let's use a named function:

```javascript
function printBook(book) {
  console.log(book);
}
books.forEach(printBook);
```

Now, calling books.forEach(printBook) is essentially equivalent to:

```javascript
printBook(books[0]);
printBook(books[1]);
...
printBook(books[books.length - 1]);
```

### Map

I hope you agree that the forEach loop improved our code on multiple levels. However, we can observe a much more spectacular improvement with the following example. Let's have a look at the following use case: _given an array of post ids, return an array of post objects_. Let's assume that we have a REST API available that will return a post object given a post id (we will use an actual API from [https://jsonplaceholder.typicode.com](https://jsonplaceholder.typicode.com)).

```javascript
var webApiUrl = "https://jsonplaceholder.typicode.com/posts/";
var postIds = [1, 4, 5];
var postPromises = [];

for (var i = 0; i < postIds.length; i++) {
  var promise = fetch(webApiUrl + postIds[i]).then(response => response.json());
  postPromises.push(promise);
}

Promise.all(postPromises).then(posts => console.log(posts));
```

In the above piece, we iterate over the array of ids. For each id we call the REST API using [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) and get a promise object in return. We push this object to a special array postPromises  where we store the results. Finally, we call Promise.all  which would transfer the array of promises into a single promise containing the desired array of posts (_if you're not familiar with promises, here is a [good read](https://developers.google.com/web/fundamentals/getting-started/primers/promises))._ That seems like a lot of code to do such a simple thing! And we have even more "imperative code overhead" - not only we need to maintain the index variable but we also have to create an array to which we will add results of the fetch  call, one by one. Let's now see a functional solution to the same problem.

```javascript
var webApiUrl = "https://jsonplaceholder.typicode.com/posts/";
var postIds = [1, 4, 5];

var postPromises = postIds.map(postId => fetch(webApiUrl + postId).then(response => response.json()));

Promise.all(postPromises).then(posts => console.log(posts));
```

So what does map  do? It takes an array and transforms it to another array by applying given function on each element. Let's see how it is different from forEach:

*   forEach  runs given function on each element
*   map  runs given function on each element and collects the returned values, producing a new array

![](/images/2017/08/Map-explained.png "Map explained")

### Filter

So far the functions we've discussed operated on all elements of an array. What if we would like to apply some operation only to some elements of an array? In other words, what if we wanted to filter out some elements? As usual, let's see an example of an imperative approach first. We will extend the example from the previous post: _given an array of post ids, return an array of posts _**_but only for even ids_**. In the imperative version, we can simply filter out odd ids by adding an if  statement:

```javascript
var webApiUrl = "https://jsonplaceholder.typicode.com/posts/";
var postIds = [1, 4, 5];
var postPromises = [];

for (var i = 0; i < postIds.length; i++) {
  if (postIds[i] % 2 == 0) {
    var promise = fetch(webApiUrl + postIds[i]).then(response => response.json());
    postPromises.push(promise);
  }
}

Promise.all(postPromises).then(posts => console.log(posts));
```

Let's now have a look at a functional version:

```javascript
var webApiUrl = "https://jsonplaceholder.typicode.com/posts/";
var postIds = [1, 4, 5];

var postPromises = postIds
  .filter(postId => postId % 2 == 0)
  .map(postId => fetch(webApiUrl + postId).then(response => response.json()));

Promise.all(postPromises).then(posts => console.log(posts));
```

We've added a call to the filter  function. The filter  function takes a **filtering function** and applies it to every element of an array. If the value returned by the filtering function is false, than given element is not included in the resulting array. In other words, the filtering function is used to decide which elements should stay and which should be filtered out. 

![](/images/2017/08/filter-explained.png "filter explained") 

In the above example, you can already see how easily we can **chain** invocations of higher order functions. This is because every such function is invoked on an array and returns an array. With such code, the **data flow** is immediately visible and easy to understand. What I mean by that is that it's easy to understand how the array is transformed. 

![](/images/2017/08/chaining-higher-order-functions-1.png "chaining higher order functions")

### Conclusion

In this post we've discussed three very useful functions which allow you to better articulate your intents with code. What's more, using these functions might result in less buggy code - they present fewer opportunities to make a bug then traditional, imperative code. The forEach  example was basic and rather theoretical while the other examples were taken from a real-world situation. In the next post, we will continue exploring the world of higher order functions by looking at  reduce. 

If you have any issues understanding anything in this post or if you simply would like to provide feedback, please leave a comment below. I want this course to be as good as possible and I need your help for that! If you found this post helpful, please consider sharing it on Facebook or Twitter.
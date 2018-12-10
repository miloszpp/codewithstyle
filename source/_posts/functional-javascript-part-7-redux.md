---
title: 'Functional JavaScript part 7: Redux'
url: 383.html
id: 383
categories:
  - JavaScript
  - Web
date: 2017-09-18 16:15:49
tags:
  - functional programming
  - redux
---

So far you had a chance to learn about two big ideas in functional JavaScript: functional array operations and immutability. Especially the latter concept could seem slightly theoretical to you. It's time to see a practical application of immutability. You will now learn about Redux - an amazing concept which will change how you think about building applications. 

**This post is part of the [Functional Programming in JavaScript series](https://codewithstyle.info/functional-programming-javascript-plain-words/).**

### Application state

Every web application in JavaScript has **state**. State is the sum of all data stored in memory during execution of the application. Every list of objects that you fetch from some backend API is part of the state. Variables that indicate whether some UI component should be visible or not are part of the state. Information about the currently logged user is part of the state. In a traditional Single Page Application the state is distributed across different places and can be updated from many different places. Therefore, it's quite easy to lose control over the state. At some point, when making another change to your application, you might be surprised to learn that this particular variable is being updated by some method that you totally forgot about.

### Redux approach to state management

Redux says that you should store the application state in a single object. It can be a complicated, deeply nested object. The important thing is that the state is stored in a single location (as opposed to being distributed across the whole application). Another rule imposed by Redux is that the state should be immutable - you should never change anything in it manually. Instead, whenever you wish to update something you should return a new copy of the state. Redux also says that all changes to the state should be initiated by **actions**. An action could be for example clicking on a button or receiving some data. The action can contain some additional data. So, you have a state and an action object. Given these two you should produce a new state object. Redux says that a function which takes a state and an action and produces a new state is called a **reducer**. 

![](/images/2017/08/reducer.png "reducer")

### Example

Let's have a look at some real example. You are working on a bookstore application. Such application would store a list of books as part of its state. Therefore, the **state** object could look like this:

```javascript
const state = {
  books: [
    { title: "Code Complete", author: "Steve McConnell", quantity: 10 },
    { title: "Clean Code", author: "Robert Cecil Martin", quantity: 4 }
  ]
};
```

One of the possible actions a user can perform is to buy a book. Let's see how an object describing such an **action** could look like.

```javascript
const action = {
 type: "BUY_BOOK",
 title: "Code Complete",
 quantity: 2
};
```

Now we need to write a **reducer** \- a function that will take the state and the action and produce a new state object. Given a list of books with quantities and an action object saying which and how many books are being bought, we need to find the book in the state and decrease its in-store quantity. Keep in mind that we need to return a new state object and not modify the existing one!

```javascript
function reduce(state, action) {
  if (action.type === "BUY_BOOK") {
    const newBooks = state.books.map(book => {
      if (book.title === action.title) 
        return { 
          ...book, 
          quantity: book.quantity - action.quantity
        };
      else
        return book;
    });
    return { ...state, books: newBooks };
  }
  else return state;
}
```

Let's go through this code. First, we inspect the action's type - there will be more actions in our application so we need to differentiate between them. Next, we map the existing collection of books to a new collection of books. The new one will be very similar to the old one except it will have decreased quantity for one of the books. That's exactly what the function passed to map does. For most of the books it will return an unchanged object. However, when it finds a book with title corresponding to the one in the action, it will produce a new book object with decreased quantity.

### Benefits of Redux

Ok, but what are the benefits of Redux? So far it looks like a complicated way to do simple things. That's actually true. It doesn't make much sense to use Redux when your application's state is very simple. It shows its merit when you have to deal with a very complicated application that would normally be implemented with state distributed in many places. In such case, an application written the Redux way would be much less prone to introducing new bugs. Representing your application's state as a single object gives you other interesting benefits. For example, you could store every action before applying it. You would then get a full history of your application's state. It would be very easy to **travel back in time** or see how would your application look like if one of the actions hasn't happened (by applying all actions apart from the one). In fact, there are [tools](https://github.com/gaearon/redux-devtools) which allow you to do exactly that.

### More on reducers

You might find the name _reducer_ sounding familiar. Indeed, reducers are very much related to the reduce  higher order function which you learned about in [one of the previous chapters](https://codewithstyle.info/functional-javascript-part-3-reduce/). Reduce would operate on an array and would take a function which accepts the existing accumulator and next element array and returns the new accumulator. That's exactly what a reducer function in Redux does! If you were given an array of actions and a reducer function than you could call reduce on that list and provide the reducer function as an argument. It's also worth mentioning that functions such as reducers have a special name in functional programming. They are called **pure functions**. A function is pure if given some specific arguments will always return the same result. It means that the result doesn't depend on any mutable state as is the case in Redux. Additionally, pure functions cannot cause any mutations themselves - in fact, they cannot produce any **side effects** at all (e.g. they cannot manipulate the DOM or print to console).

### Redux library

Wait, wasn't Redux supposed to be a library? The code above doesn't include any non-vanilla JavaScript calls. That's true - you can actually build applications the Redux way without using the [Redux library!](http://redux.js.org/) The library itself provides you with some utilities that make it easier to build applications. However, this article focuses on the concepts behind Redux and not on the library itself. Once you understood the concepts you will find learning the library very easy.

### Summary

This chapter combines lots of ideas from the previous posts and shows you a really powerful concept in functional programming. Redux is a completely new way of building applications and definitely takes some time to get accustomed to but it can make handling complexity much easier. Note the heavy use of concepts introduced in the previous parts of the code - higher order functions, object spread operator and immutability. It shows how interconnected the concepts in functional programming are. For me it feels like pieces of a puzzle coming together! If you have any issues understanding anything in this post or if you simply would like to provide feedback, please leave a comment below. I want this course to be as good as possible and I need your help for that! If you found this post helpful, please consider sharing it on Facebook or Twitter.
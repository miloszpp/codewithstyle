---
title: Combining two objects in JavaScript
tags:
  - javascript
url: 8.html
id: 8
categories:
  - JavaScript
  - Quick solutions
  - Web
date: 2016-01-11 21:38:00
---

One thing that I have to do rather often when writing JavaScript code is to combine properties from two objects into a single object. **UPDATE: This article was originally called _Combining two objects in lodash_. I've updated it to cover more ways of combining objects in JavaScript** For example, given these two objects:

    const a = {
      name: "John",
      age: 23
    };
    const b = {
      job: "Analyst"
    };
    

...what can be done to avoid copying properties manually?

    const c = {
      name: a.name,
      age: a.age,
      job: b.job
    };
    

Solution 1: `Object.assign`
---------------------------

`Object.assign` is a built-in method introduced to JavaScript as part of the ES6 standard. It allows you to copy properties from a target object to the source object. A possible solution to the above problem could be:

    Object.assign(b, a);
    

This way `b` would have both its own properties and `a`'s properties. However, we might want to avoid modifying `b`. In such case, we can introduce a new, empty object and copy properties from `a` and `b` to it.

    const c = Object.assign({}, a, b);
    

Solution 2: lodash
------------------

If for some reason you cannot use ES6 language features in your application, you can resort to using the _lodash_ library.

    const c = _.assign({}, a, b);
    

_If you'd like to learn more about lodash, check out my [free e-book about Functional Programming in JavaScript](https://codewithstyle.info/functional-programming-javascript-plain-words/)._

Solution 3: object spread operator
----------------------------------

Another ES6-based solution and my personal favourite is to use the object spread operator.

    const c = { ...a, ...b };
    

The triple-dot operator _unwraps_ an object and lets you put its properties into a new object. By _unwrapping_ both `a` and `b` and putting it into a new object (`{}`) we end up with an object having both `a`'s and `b`'s properties. So, which way is the best way? It depends on your preference and requirements. Just pick one and be consistent in your choice!
---
title: 'Functional JavaScript part 6: immutable.js'
url: 387.html
id: 387
categories:
  - JavaScript
  - Web
date: 2017-08-24 14:33:27
tags:
  - functional programming
  - immutability
---

In the previous post we saw some techniques that can help us implement immutability. However, none of them feel very natural in plain, vanilla JavaScript. It doesn't feel like the language and the standard library have been designed with immutability in mind. It's not surprising given that one of the language's main purposes was to **mutate** DOM (the Document Object Model). 

![](/images/2017/08/Zrzut-ekranu-2017-08-24-o-21.54.50-300x70.png) 

Fortunately, [Immutable.js](https://facebook.github.io/immutable-js/) comes to rescue. It is a library which makes working immutable objects much more natural. In this post we will learn how to use the library and how we can benefit from it. 

**This post is part of the [Functional Programming in JavaScript series](https://codewithstyle.info/functional-programming-javascript-plain-words/).**

### Map instead of object

Immutable.js introduces a few immutable data structures. Let's focus on Map  which is an immutable version of plain JavaScript object. The name comes from the world of algorithms and data structures where we use it as a name for a structure which _maps_ some keys to some values (just like a JavaScript object). It's very easy to initialize a Map from a plain JavaScript object:

```javascript
const product = Immutable.Map({ name: "Product", quantity: 10 });
```

We must not use assignment operator with a Map object's properties. Instead, we should call the set  method on it.

```javascript
const product = Immutable.Map({ name: "U-lock", quantity: 10 });
const updatedProduct = product.set("category", "safety");
```

As expected set will not mutate the original object. Instead, it will return a new updated object. We can easily return to the world of plain JavaScript objects with toJS:

```javascript
updatedProduct.toJS();
```

### Nested objects

There are many interesting operations available on Map. For example, setIn works great with deeply nested objects. Imagine having an object with several levels of nesting and having to update a property residing in a deeply nested object.

```javascript
const employee = {
  name: "John",
  reportsTo: {
    name: "Alice",
    reportsTo: {
      name: "Bob"
    }
  }
};

const immutableEmployee = Immutable.fromJS(employee);
const updatedEmployee = immutableEmployee.setIn(["reportsTo", "reportsTo"], "Celine");
console.log(updatedEmployee.toJS());
```

First of all, note that we've used fromJS  instead of Map . This is because Map only works shallowly - when it sees a property that's a reference to an object it doesn't convert it to Map but jest leaves it as it is. On the other hand fromJS  will always traverse all levels of nesting. As you can see setIn takes an array of strings defining the path of properties leading to the value we are interested in. The second argument is the value we want to set the property to. Obviously setIn doesn't mutate the object but returns a fresh copy instead.

### Lists

We haven't yet discussed immutability of arrays. In the previous chapters we've talked about how to avoid mutating objects. What about arrays? Basic array operations in vanilla JavaScript are mutable - they update the array in-place:

```javascript
const numbers = [1, 2, 3, 4];
numbers.push(5);
```

An immutable version of push would have to clone the array first and add the element to the new copy. We've already discussed a similar approach when talking about sort in the [chapter about lodash](https://codewithstyle.info/functional-javascript-part-4-lodash/).

```javascript
function immutablePush(arr, newElement) {
  const arrCopy = arr.slice(0);
  arrCopy.push(newElement);
  return arrCopy;
}
```

The ES6 spread operator lets us do this in a more concise way. It's like the object spread operator - it "unwraps" array elements and lets you put them directly in another list.

```javascript
const newNumbers = [...numbers, 7];
```

But how about adding an element in the middle of an array? It gets a little messy.

```javascript
const newNumbers = [...numbers.slice(0, 2), 7, ...numbers.slice(3)];
```

Immutable.js simplifies immutable array operations by introducing a List object. List has methods similar to array's but all of them are immutable and return a fresh copy instead of mutating the list.

```javascript
const numbersList = Immutable.List([1, 2, 3, 4, 5]);
const newNumbersList = numbersList.insert(2, 99);
```

List has many useful methods which make working with it in immutable way easier than working with arrays.

### Structural sharing

We haven't touched on an important drawback of immutability yet - performance. Cloning objects in JavaScript is expensive. With immutability we have to use cloning a lot since we aren't allowed to mutate objects. This is especially painful if you have to deal with large objects and only want to update a single property. In such case you would need to re-create the whole object anyway. Fortunately, Immutable.js tries to address the issue by using various optimization techniques including **structural sharing**. It stores Maps in such a way that when you update some property you get a reference to a new Map which shares some of the data with the old Map. Since Maps are immutable this isn't a problem - we don't need to worry that the shared part will be accidentaly modified.

### Benchmark

I have created a benchmark in order to measure the performance of Immutable.js. In this benchmark I create an array of 100000 moderately complex objects. Next, I compare the performance of changing a single property of this object in one of 3 ways:

1.  Simply mutate the object (the mutable way) - **224 test runs/second**
2.  Deep clone using spread operator - **2.36 test runs/second**
3.  Clone using Immutable.js - **4.97 test runs/second**

We can see that all immutable options are much, much slower than simply mutating the object. This is because of the cost of cloning objects. However, we can also see that cloning with Immutable.js is two times faster than using the spread operator. The benchmark does not take into consideration the fact that in order to use Immutable.js our array of objects had to be converted into an array of Maps which is a quite expensive operation. The results would be much worse in such case. Therefore, it makes sense to use Immutable.js all the way throughout your application in order to avoid converting between Immutable.js data structures and vanilla JavaScript data structures. The benchmark is publicly available. Feel free to play with it: [https://jsperf.com/immutability-performance-nested-objects/1](https://jsperf.com/immutability-performance-nested-objects/1)

### Summary

This post was a quick introduction to Immutable.js. We've learned about immutable data structures such as Map and List, how to perform operations on them and how Immutable.js approaches performance issues related to immutability. It's understandable if the posts about Immutability seem a little abstract so far. In the next post we will fix it - you will see how useful can it be on an example of Redux. If you have any issues understanding anything in this post or if you simply would like to provide feedback, please leave a comment below. I want this course to be as good as possible and I need your help for that! If you found this post helpful, please consider sharing it on Facebook or Twitter.
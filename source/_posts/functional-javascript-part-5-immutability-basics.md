---
title: 'Functional JavaScript part 5: immutability basics'
url: 376.html
id: 376
categories:
  - JavaScript
  - Web
date: 2017-08-17 20:32:59
tags:
---

In previous posts we've discussed how to deal with arrays in a functional way. We've learned about an important concept in Functional Programming: higher order functions. Let's now tackle another very important concept: immutability. **This post is part of the [Functional Programming in JavaScript series](https://codewithstyle.info/functional-programming-javascript-plain-words/).**

### What is immutability?

Immutability is a fancy name for a specific rule for writing code. This rule says: _never change a value or reference once it has been assigned_. There are many reasons why you might decide to decide to use immutability. Most importantly, it makes your code easier to reason about. When working with traditional code with mutations, every time you call some function and pass an object to it, you have to assume that the function might change some property in the object you've passed. Such changes might be surprising, especially when some other developer is working on that function.

#### Example of issues with mutability

If you decide to embrace immutability in your code base you can be sure that if you pass an object to a function, none of its properties will be changed. Less surprises and less possibilities of error. Let's see an example:

function setShipmentAddress(person, product) {
  product.shipmentAddress = person.address;
}

Assume that we are using this function somewhere in our code. It's part of a common library and another developer is responsible for it so we don't bother looking at the source code. One day, the developer is told that the product's address should always be in upper case. Let's assume he's not terribly careful and decides to implement it this way:

function setShipmentAddress(product, person) {
  person.address = person.address.toLowerCase();
  product.shipmentAddress = person.address;
}

You don't bother to look at the source code and start using the function straight away. Initially, it looks good - the function has assigned a lowercase address to product. However, after deployment to production you start receiving bug reports - why is person's address displayed in lower case? Oh, wait...

#### Immutable solution

All this mess could be avoided if you agreed on the immutability rule. In such case the other developer would have to implement setShipmentAddress in such a way that it would not mutate any of the input objects. Instead, it would return a fresh product object with updated address. You could then assume that this function will never change either product  nor person so the above situation would never happen!

function setShipmentAddress(product, person) {
  return {
    name: product.name,
    quantity: product.quantity,
    shipmentAddress: person.address
  };
}

You can see that with immutability we have a clear separation of function's inputs and outputs. This is a very simplified example. In reality such **unintentional side effects** can be much more subtle and have much worse consequences. Let's see how to avoid them at all thanks to immutability.

### Constants

We will start small and have a look at simple variables. Variables in JavaScript are obviously mutable - once you assign an object or value to a given variable, you are free to change it at any point in the future. ES6 introduced a new keyword to the language: const. It is meant to be used in place of the var keyword. Declaring a variable as const  means that you cannot change it in the future!

const a = 10;
a = 5;

Running the above code results in an error:

Uncaught TypeError: Assignment to constant variable.

You might be wondering why anyone would use one such thing. The main advantage is that it helps you avoid situations in which you accidentally change an existing variable. What's more, it expresses your intentions better. If you mark a variable as const and then another developer comes and wants to change your code in a way that would require reassignment on that variable, they will realize that wasn't your intention. They will think twice before making the change.

#### Example

Let's look at a code piece from one of the previous chapters:

var safetyProducts = _.filter(products, p => p.category === "Safety");
var quantities = _.map(safetyProducts, p => p.quantity);
var bigQuantites = _.filter(quantities, q => q > 10);

Here we declare three variables and each of them is used only once. It will never make sense to re-assign to them. Therefore, it makes sense to change them to const.

const safetyProducts = _.filter(products, p => p.category === "Safety");
const quantities = _.map(safetyProducts, p => p.quantity);
const bigQuantites = _.filter(quantities, q => q > 10);

Note that this code piece was already written in functional style. Using const  often makes sense when dealing with functional code. One final remark about const is that while it guarantees that the variable cannot be changed, it doesn't say anything about the object (or array) assigned to that variable. Therefore, this code would be perfectly legal:

const product = {
  name: "U-lock",
  quantity: 10
}
product.quantity = 15;

This is because the variable stores a reference to an object. We only mark the reference as immutable but not the object.

### ![](/images/2017/08/const-and-references-1.png "const and references")

### Freezing objects

How do we implement immutable objects in JavaScript then? We can use the Object.freeze method. It's job is to mark all properties as read-only and prevent adding new properties.

const john2 = {
  name: "John",
};
Object.freeze(john2);
john2.age = 33;
console.log(john2); // prints {name: "John", age: 25}

As we can see the age property was **not** changed. If we enabled strict mode this code would throw an error.

### Don't mutate, clone

Ok, but how can we implement "changes" to frozen object? Instead of mutating it we will simply return a new copy with applied changes. How to implement this, though? One of the ways would be:

function buyProduct(product) {
  return {
    name: product.name,
    quantity: product.quantity - 1
  };
}

However, imagine having an object with 50 properties. We would need to rewrite them all!

#### Object.assign

Let's have a look at some more clever ways to do this. The first option is to use the Object.assign  method introduced in ES6. Object.assign takes a target object and a source object(s) and copies all properties from the source object(s) to the target object.

const product = {
  name: "U-lock",
  quantity: 10
};
Object.assign(product, { category: "Safety" });
console.log(product);

This code will print the following result to the console:

{name: "U-lock", quantity: 10, category: "Safety"}

Wait, but we are mutating the product object here, right? That's correct. However, we can use a little trick to return a new object instead of mutating the existing one. We can easily specify a new, empty object as the source object:

const categorizedProduct = Object.assign({}, product, { category: "Safety" });

Here we are providing two source objects and an empty target object. All of the properties from the source objects will be copied to the target empty object. As a result we get a fresh object with all the properties of product and the new category  property. It's important to note that Object.assign  performs **shallow** assignment. If we have a nested object in one of the sources than it will **not** be cloned. Is it good or bad? It depends on our use case.

const john = {
  name: "John",
  age: 25,
  address: {
    street: "Wynalazek",
    city: "Warsaw"
  }
};

const alice = Object.assign({}, john, { name: "Alice" });
alice.address.city = "London";

console.log(john.address.city); // "London"

#### Object spread operator

This is actually not part of the JavaScript specification (yet). It is part of a [proposal](https://github.com/tc39/proposal-object-rest-spread) and will likely make it to one of the future JavaScript versions. However, it's already supported in most modern browsers. Object spread operator is an even more convenient way of applying changes to objects without having to mutate them. Let's see an example:

const alice2 = { ...john, name: "Alice" };

This new syntax can be roughly translated into: take all properties from john object, combine them with the new name property and put it all in the new object. The three dots are applied to an object in order to "unwrap" its properties. Since such "unwrapped" properties cannot live on their own, they should always be put within curly braces. But curly braces always create a new object hence we will get a fresh object with copied properties. We don't need to add any new property. This line would simply clone john :

const johnCopy = { ...john };

Just as in the case of Object.assign  we have to pay special attention to nested objects.

### Summary

In this post we've discussed four techniques which can help us write immutable code: constants, Object.freeze, Object.assign  and object spread operator. The last two techniques do not enforce immutability but rather make it easier to implement. I hope you agree with me that there are some benefits to using immutability. If you're still not convinced, bear with me until we talk about Redux which unleashes the full potential of immutability. Before that, we will take a look at Immutable.js - a library which can make writing immutable code feel much more natural. If you have any issues understanding anything in this post or if you simply would like to provide feedback, please leave a comment below. I want this course to be as good as possible and I need your help for that! If you found this post helpful, please consider sharing it on Facebook or Twitter.
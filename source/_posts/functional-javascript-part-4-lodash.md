---
title: 'Functional JavaScript part 4: lodash'
url: 361.html
id: 361
categories:
  - JavaScript
  - Web
date: 2017-08-12 10:17:25
tags:
---

With this post we will finalize the part of this course devoted to array operations Vanilla JavaScript provides us with some higher order functions (such as forEach , filter , map and reduce). However, there are actually many more such functions in the world of functional programming. [Lodash](https://lodash.com/) is a library which extends your arsenal of higher order functions. Let's have a look at how to use it in practice. **This post is part of the [Functional Programming in JavaScript series](https://codewithstyle.info/functional-programming-javascript-plain-words/).**

### Exploring lodash

The best place to explore functions available in lodash is the [documentation](https://lodash.com/docs/). You will notice that there are many expandable sections on the left hand side. For now let's focus on **Array** and **Collection**. ![](http://codewithstyle.info/wp-content/uploads/2017/08/lodash_documentation-1024x612.png)

### Using lodash

Lodash methods are not available directly on the array object. It could be achieved with JavaScript's prototypical inheritance but it's not considered a good practice to extend native prototypes (actually, it's disputable but [lodash creators decided not to do this](https://github.com/lodash/lodash/issues/409)). Therefore, in order to use a lodash method we need to call it on the global _  object. You may note that this will make chaining less convenient but there is a cure for that - we'll look at it at the end of this post. Let's see a usage example:

var numbers = \[1, 2, 3, 4, 5\];
console.log(_.drop(numbers, 2));

Drop function takes an array and a number of elements to drop. It returns a new array that doesn't contain the first n  elements. As a side note, it's actually not a higher order function since it doesn't take a function as argument.

### Validation with every and any

Let's consider the following requirement: _we're running a bike parts shop. W__e are given a list of items in the customer's shopping cart. We should validate that he is ordering at least one piece of each item._ All of the imperative approaches to this problem I can think of are a little clumsy. We could either:

*   count the items with quantity equal to 0 and check if the number is equal to zero
*   have a separate function in which we iterate over the items and return early if we encounter one with quantity equal to 0

Let's see how we can use lodash in order to solve it in an elegant way:

var basket = \[
  { name: "Cable Lock", quantity: 2 },
  { name: "U Lock", quantity: 5 },
  { name: "Tail Light", quantity: 0 }
\];

if (!_.every(basket, item => item.quantity > 0)) {
  alert("You must order at least 1 piece of each item");
}

Function every takes a function that evaluates to true or false, applies this function on all elements and returns true only if the function was true on all elements. In other words, it checks whether **every** element satisfies given condition. In fact, what we want to do is to check if it is not true that every element satisfies given condition. Therefore, we can check whether there are **some** elements that don't satisfy our condition (the condition being quantity greater than 0).

if (_.some(basket, item => item.quantity == 0)) {
  alert("You must order at least 1 piece of each item");
}

Which results in even cleaner and more readable code. Doesn't it feel a bit like writing in natural language?

### Grouping with groupBy

Here comes another requirement! _We've received a list of available products from some backend API. It would be nice to display them in separate boxes based on the category they belong to._ The imperative solution is particularly verbose:

var products = \[
 { name: "Cable Lock", category: "Safety" },
 { name: "U Lock", category: "Safety" },
 { name: "Tail Light", category: "Basics" }
\];

var productsByCategory = {};
for (var i = 0; i < products.length; i++) {
  var category = products\[i\].category;
  if (!productsByCategory.hasOwnProperty(category))   {
    productsByCategory\[category\] = \[\];
  }
  productsByCategory\[category\].push(products\[i\]);
}
console.log(productsByCategory);

We create an empty object (productsByCategory) in which we will store the results - keys will represent categories and for each key we will store an array of products. Next, we iterate over the products. For each product we check whether we already have an entry in the productsByCategory object. If we don't then we need to create it and initialize it with an empty array. Finally, we add the product to the list under its category. The result will look like this:

{Safety: Array(2), Basics: Array(1)}

I bet you're expecting the functional version to be much simpler - and it is. Yes, it's a single line again.

var productsByCategory = _.groupBy(products, product => product.category);

The function takes an array and a function which determines how to group the elements of that array. The grouping functions is evaluated for each element. Those elements for which the same value is returned are packed into separate groups. Finally, an object is returned with keys equal to unique values returned by the grouping function applied on all of the elements. ![](/images/2017/08/drawit-diagram.png "drawit diagram") The grouping function does not have to be a simple property selector - we can put any sort of expression in it. For example, it's trivial to split products into groups based on the length of their names:

var productsByNameLength = _.groupBy(products, product => {
  if (product.name.length < 10) return "short";
  if (product.name.length >= 10 && product.name.length < 20) return "medium";
  return "long";
});

### Sorting with orderBy

The last useful function I'd like you to look at is orderBy. As the name suggests, it let's you sort an array. Vanilla JavaScript already has a sort  function built-in. orderBy  is more convenient to use and more functional in its nature. Firstly, while sort  orders the array in-place, orderBy  doesn't modify the existing array but returns a fresh copy. Secondly, sort takes a comparator function - a function which takes two elements and compares them. orderBy  is more consistent with other functions we've looked at - it takes a function which selects the value to use for ordering. Let's say that we would like to have a fresh copy of our products array sorted by quantity. Without lodash we would need to do the following:

var sortedBasket = basket.slice(0);
sortedBasket.sort((a, b) => a.quantity - b.quantity);
console.log(sortedBasket);

In the first line we use slice  in order to clone the array. slice  is for cutting out slices of an array. We can tell it to cut out the whole array as a slice which will give as a copy of this array. Next we call sort  providing a comparator function. This comparator takes two elements and returns a negative number if the first element is smaller than the second, positive number if the second element is smaller and 0 if the elements are equal. By subtracting the second element from the first we will achieve the desired behavior. Let's now see a lodash solution:

var sortedBasket = _.orderBy(basket, item => item.quantity);

Here we just specify that we want to use the quantity property for sorting. The original array will remain as it was. You might be wondering why it can be important to not change the original array. One important aspect of functional programming is **immutability**. The idea is to never modify existing objects but instead return new copies. It might sound wasteful but in fact it has many advantages. We will take a deeper look at immutability in the posts to come. For now, here are a few practical examples of when it's required to not sort an array in-place.

*   One example would be passing an array between two Angular components - we don't want one of the components to modify internal state of another component
*   Another example is Redux where we are not allowed to mutate state as we always have to return a fresh copy of the state; there will be a separate chapter dedicates solely to Redux

### Chaining

I've already mentioned that with lodash syntax it's no longer as easy to chain method calls as with vanilla Javascript. However, lodash has solved this problem. Let's see an example:

var safetyProducts = _.filter(products, p => p.category === "Safety");
var quantities = _.map(safetyProducts, p => p.quantity);
var bigQuantites = _.filter(quantities, q => q > 10);

We have three consecutive lodash calls here. It would be nice to chain these calls so that the data flow would be more readable.

var bigQuantites = _.chain(products)
  .filter(p => p.category === "Safety")
  .map(p => p.quantity)
  .filter(q => q > 10)
  .value();

Here we use the chain  method to wrap our array in a special object that knows about all the lodash methods. Next, we can simply call lodash methods directly on this object. At the end we need to unwrap our array by calling value.![](/images/2017/08/lodash-chaining.png "lodash chaining")

### Summary

And that's all about lodash. Of course, there is much, much more in it and I strongly encourage you to explore it using the documentation. Since you already know the concept of higher order functions understanding the remaining functions will be much easier for you. I hope you find higher order functions on arrays useful and fun. They're really worth learning and understanding since the concept is very widespread. The investment will pay off in random places (for example when learning RxJS). How to use functional array operations? From now on, every time you find yourself writing a for loop, stop for a moment and try to figure out if you can write it using in a more functional style. The cases when it's not possible or beneficial are very, very rare. This post concludes the chapter about array operations. Next we will take a look at immutability in JavaScript! If you have any issues understanding anything in this post or if you simply would like to provide feedback, please leave a comment below. I want this course to be as good as possible and I need your help for that! If you found this post helpful, please consider sharing it on Facebook or Twitter.
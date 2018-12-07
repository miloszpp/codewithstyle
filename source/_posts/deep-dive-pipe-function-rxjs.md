---
title: Deep dive into pipe function in RxJS
url: 723.html
id: 723
categories:
  - JavaScript
  - Uncategorized
  - Web
date: 2018-07-02 09:00:08
tags:
---

Version 5 of RxJS introduced the concept of _lettable_ (also known as _pipeable_) operators. Version 6 went one step further and deprecated the old way of calling operators (method chaining). You might have already used the `pipe` function. But do you really understand what it does? ![](https://codewithstyle.info/wp-content/uploads/2018/06/Monads-part3-—-kopia.png)

Composing functions
-------------------

RxJS is often called a **functional-reactive programming** library. It should not come as a surprise that you will find many functional programming inspirations in it. One of them is the `pipe` function. Take a look at the below piece of code:

    const getElement = 
        (id) => document.getElementById(id);
    
    const getValue = 
        (element) => element.value;
    
    function logElementValue(id) {
      const el = getElement(id);
      const value = getValue(el);
      console.log(value);
    }
    

The `logElementValue` function takes an `id` and logs to the console the value of the element with provided `id`. Can you see a pattern in this function's implementation? Firstly, it calls `getElement` with `id` and stores the result in `el`. Next, the result is passed to `getValue` which produces a new result, `el`. Finally, `el` is passed to `console.log`. What this function does is simply taking the result of a function and passing it as an argument to another function. Is there a better, more concise way to implement this function? Let's say we just have two functions (`getElement` and `getValue`). We will implement a generic function called `compose` that will pass the result of `getElement` to `getValue`.

    const compose = (f, g) => x => g(f(x));
    

The definition is very simple but may take a moment to parse. We've defined a function that takes two functions `f` and `g` (that would be `getElement` and `getValue` in our case) and returns a new function. This new function will take an argument, pass it to `f` and then pass the result to `g`. That's exactly what we need! Now I can rewrite `logElementValue`:

    function logElementValue(id) {
      const getValueFromId = compose(getElement, getValue);
      const value = getValueFromId(id);
      console.log(value);
    }
    

How about more than two functions?
----------------------------------

But, wait! Once we have the result of calling `getValueFromId` we immediately pass it to `console.log`. So it's the same pattern here. We could write it like this:

    function logElementValue(id) {
      const getValueFromId = compose(getElement, getValue);
      const logValue = compose(getValueFromId, console.log);
      logValue(id);
    }
    

But life would be much simpler if `compose` could take any number of functions. Can we do this? Sure:

    const composeMany = (...args) => args.reduce(compose);
    

Another brain teaser! `composeMany` takes any number of functions. They are stored in `args` array. We `reduce` over `args` composing every function with the result of composing previous functions. Anyway, the results is a function that takes any number of functions and will pass the result of `N-th` function to `(N+1)-th` function. But what have we achieved by that?

    function logElementValue(id) {  
      const logValue = composeMany(getElement, getValue, console.log);
      logValue(id);
    }
    

Which can be simplified even more:

    const logElementValue = composeMany(getElement, getValue, console.log);
    

Isn't that cool? We have significantly simplified the code. It's now very clear what `logElementValue` does. And by the way - `composeMany` is just a name a came up with. The official name is `pipe`!

    const logElementValue = pipe(getElement, getValue, console.log);
    

Back to RxJS
------------

Let's take an example of `pipe` usage in RxJS.

    number$.pipe(
        map(n => n * n),
        filter(n => n % 2 === 0)
    );
    

We can also write it in a different way:

    const { pipe } = rxjs;
    
    const transformNumbers = pipe(
         map(x => x * x),
         filter(x => x % 2 === 0),
    );
    
    transformNumbers(number$).subscribe(console.log);
    

And the result is exactly the same! As you can see, the `pipe` function in RxJS behaves in exactly the same way that the `pipe` function that we've defined in the first part of the article. It takes a number of functions and composes them by passing the result of a function as an argument to another function. You might say that this is different than the previous example because here we're invoking `map` and `filter` and not simply passing them. Actually, both `map` and `filter` will return functions. We're not composing `map` and `filter` themselves but rather the functions returned by invoking them. You can check out how RxJS implements `pipe` function [here](https://github.com/ReactiveX/rxjs/blob/94156a214f905555b6e57bc3f7cf965629028406/src/internal/util/pipe.ts).

Pipeline operator
-----------------

Our function is such a useful concept that it might be added as a separate operator to the JavaScript language! It would mean that the example from the previous article can be written in an even simpler way:

    const logElementValue = getElement |> getValue |> console.log;
    

You can see the details of the proposal [here](https://github.com/tc39/proposal-pipeline-operator).

Summary
-------

I hope this article helped you understand what `pipe` function is all about. You should now feel more comfortable using it! The fact that RxJS migrated from the traditional, object-oriented approach of applying operators to the pipeline approach shows how strong the influence of functional programming is nowadays. I think that's great! Let me know in comments if you prefer `pipe` function to traditional method chaining. \[yikes-mailchimp form="1" description="1"\]
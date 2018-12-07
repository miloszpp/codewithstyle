---
title: 'Advanced functional programming in TypeScript: monads and generators'
url: 638.html
id: 638
categories:
  - JavaScript
  - TypeScript
date: 2018-03-19 08:00:27
tags:
---

Welcome to the second post in the series. [In the first one](https://codewithstyle.info/advanced-functional-programming-in-typescript-maybe-monad), you had a chance to build your first monad in TypeScript. In this post, you'll see how to take advantage of **generators** to make the monadic code more readable. ![](https://codewithstyle.info/wp-content/uploads/2018/03/Monads-part2.png) **You can find all the code from the series in [this repository](https://github.com/miloszpp/typescript-monads). Check out different branches for code relevant to the specific part of the series.**

Generator functions
-------------------

Generator functions have been introduced to JavaScript [as part of the ES6 standard](http://es6-features.org/#GeneratorControlFlow). They are a special case of functions where it's possible to pause execution in the middle of the function body. This might sound counter-intuitive, especially if you consider the fact that JavaScript is single-threaded and follows the _Run-to-completion_ approach. However, with generators, the code is still executing synchronously. **Pausing execution** means giving the control back to the caller of the function. The caller can then resume execution at any point. Let's see a simple example:

    function* numbers(): IterableIterator<number> {
        console.log('Inside numbers; start');
        yield 1;
        console.log('Inside numbers; after the first yield');
        yield 2;
        console.log('Inside numbers; end');
    }
    

There are two pieces of new syntax here. Firstly, there is a `*` following the `function` keyword. It indicates that `numbers` is not a regular function but a **generator function**. Another new thing is the `yield` keyword. It's a bit like `return` but it can be used multiple times inside the function's body. By yielding a value, the generator returns a value to the caller. However, unlike `return`, the caller may decide to resume execution and give control back to the function. When it happens, the execution continues from the latest `yield`. First, we need to invoke `numbers` to get a generator instance.

    const numbersGenerator = numbers();
    

At this point, not a single line of the `numbers` function has been executed. For this to happen, we need to call `next`. It will start executing the function until it encounters the first `yield`. At this point, the control will be given back to the caller.

    console.log('Outside of numbers');
    console.log(numbersGenerator.next());
    console.log('Outside of numbers; after the first next');
    console.log(numbersGenerator.next());
    console.log('Outside of numbers; after the second next');
    

When we run these lines, we will get the following output:

    Outside of numbers
    Inside numbers; start
    {value: 1, done: false}
    Outside of numbers; after first next
    Inside numbers; after the first yield
    {value: 2, done: false}
    Outside of numbers; after the second next
    

As you can see, the execution jumps back and forth between `numbers` and its caller. Each `next` call returns an instance of `IteratorResult` which contains the yielded value and a flag `done` which is set to `false` as long as there is more code to execute inside `numbers`. ![](https://codewithstyle.info/wp-content/uploads/2018/02/Generators.png) Generators are a very powerful mechanism. One of their uses is building lazy, infinite sequences. Another one is co-routines - a concurrency model where two pieces of code can _communicate_ with each other. \[yikes-mailchimp form="1" description="1"\]

Maybe implemented with generators
---------------------------------

As a reminder, we implemented the `getSupervisorName` function in the following way:

    function getSupervisorName(maybeEnteredId: Maybe<string>): Maybe<string> {
        return maybeEnteredId
            .flatMap(employeeIdString => Maybe.fromValue(parseInt(employeeIdString)))
            .flatMap(employeeId => repository.findById(employeeId))
            .flatMap(employee => employee.supervisorId)
            .flatMap(supervisorId => repository.findById(supervisorId))
            .map(supervisor => supervisor.name);
    }
    

Obviously, the code is more readable than the original version which involved three levels of nesting. However, it looks very different from regular, imperative code. Can we make it look more like imperative code? As we know, generators allow us to pause execution of a function so that it can be later resumed. This means that we can _inject_ some code to be executed between `yield` statements of given function. We could try to write a function that takes a **generator function** (a function with some `yield` statements) and injects the logic of handling empty results. Let's start by writing an implementation of `getSupervisorName` that will use `yield` statements to handle empty results.

    function* () {
        const enteredIdStr = yield maybeEnteredId;
        const enteredId = parseInt(enteredIdStr);
        const employee = yield repository.findById(enteredId);
        const supervisorId = yield employee.supervisorId;
        const supervisor = yield repository.findById(supervisorId);
        return Maybe.some(supervisor.name);
    }
    

Let's assume that `maybeEnteredId` is defined in the function's clojure and that it's type is `Maybe<string>`. Now, the semantics of `const enteredIdStr = yield maybeEnteredId` is:

*   if `maybeEnteredId` contains a value then assign this value to `enteredIdStr`
*   otherwise, the whole function should return `None`

In other words, `yield` works exactly like `flatMap`, but the syntax is very different. It's much more familiar to an imperative programmer.

Implementing `run`
------------------

This is not over yet. We still need a function that will be able to consume this generator function. In other words, we need something that will give meaning to all these `yield` statements. We'll call this function `run`. It will take a generator function and produce `Maybe` instance containing the result of the computation. Let's start with an imperative implementation of `run`:

    static run<R>(gen: IterableIterator<Maybe<R>>): Maybe<R> {
        let lastValue;
        while (true) {
            const result: IteratorResult<Maybe<R>> = gen.next(lastValue);
            if (result.done || result.value.value === null) {
                return result.value;
            }
            lastValue = result.value.value;
        }
    }
    

1.  The `run` function accepts a generator function `gen`. This function describes our computation.
2.  We enter an infinite loop and call `gen.next(lastValue)`. This call will cause control flow to enter `gen` and execute until the first `yield` (ignore `lastValue` for now).
3.  Once this happens, the control flow returns to `run`. The value passed to `yield` is wrapped inside `IteratorResult` and returned as a value of `gen.next`,
4.  `result` has a `done` flag. It indicates whether control flow inside `gen` has reached the end of its body (i.e. whether there's more code to execute).
5.  `result.value` holds the value returned by `yield`. It's an instance of `Maybe`. Therefore, we check if it's an empty result (a `None`). If this is the case, we `return` a `None` from the whole computation.
6.  Otherwise, we _unwrap_ our `Maybe` and assign the inner value to `lastValue`.
7.  The loop repeats. However, this time `lastValue` is not empty. It will be passed to `gen` as a result of calling `yield`.

There's quite a lot going on here. A good way to understand this is to think about two pieces of code communicating with each other.

*   `gen` _sends_ a `Maybe<T>` instance to `run` by calling `yield m`
*   `run` _replies_ with an instance of `T` by calling `gen.next(lastValue)`

The whole point of this is that we can hide the logic of empty result handling inside `run`. From the caller's perspective, it will look like this:

    function getSupervisorName(maybeEnteredId: Maybe<string>): Maybe<string> {
        return Maybe.run(function* () {
            const enteredIdStr = yield maybeEnteredId;
            const enteredId = parseInt(enteredIdStr);
            const employee = yield repository.findById(enteredId);
            const supervisorId = yield employee.supervisorId;
            const supervisor = yield repository.findById(supervisorId);
            return Maybe.some(supervisor.name);
        }());
    }
    

What we've achieved is a clean and elegant implementation of `getSupervisorName` with all the boilerplate hidden in `run`. However, contrary to the `flatMap` solution, this code looks more intuitive to an imperative programmer.

Back to monads
--------------

That's nice, you might say, but what does it have to do with monads? We're not taking advantage of the fact that `Maybe` is a monad. Let's fix that. You might've noticed some similarity between `run` implementation and `flatMap`. Both implementations have to deal with empty results and apply a similar logic: if a `Maybe` instance is empty then return early with a `None`. Otherwise, continue the computation with the unwrapped value. It turns out that we can implement `run` using `flatMap`!

    static run<R>(gen: IterableIterator<Maybe<R>>): Maybe<R> {
        function step(value?) {
            const result = gen.next(value);
            if (result.done) {
                return result.value;
            }
            return result.value.flatMap(step);
        }
        return step();
    }
    

This recursive implementation is much more in the spirit of functional programming.

1.  We define `step` function which takes an optional `value` and passes it to `gen.next`. As we know, this will cause execution to resume inside `gen`, up to the nearest `yield`.
2.  We investigate `result.done`. If it's false (there is still some code to execute), we simply call `flatMap` on `result.value` and recursively pass `step` as _continuation_ function. `flatMap` will take care of an empty result.
3.  As long as we're not dealing with a `None`, the recursive call to `step` will run `gen` up until the next `yield`. And so one, and so one.

The client code looks exactly the same.

Summary
-------

In this post, we've looked at how generators can be leveraged to improve the experience of using monads. They make monadic code even cleaner and, what's important when working in teams, easier to understand to programmers with imperative background (i.e. the vast majority). The idea of using generators in such a way is the basis of the `async/await` mechanism. As you'll learn in the future posts, `Promise` is also a monad and `async/await` is nothing more than a specialized variant of `function*/yield`. But let's not jump ourselves ahead :) Did you find this approach interesting? What is a more readable way to write monadic expressions - with or without generators? **Share your thoughts in comments below!**
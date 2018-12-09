---
title: 'Advanced functional programming in TypeScript: functional exceptions'
url: 670.html
id: 670
categories:
  - JavaScript
  - TypeScript
date: 2018-05-03 16:01:12
tags:
  - functional programming
  - monads
  - typescript
---

After looking into different ways of implementing `Maybe` in [part 1](https://codewithstyle.info/advanced-functional-programming-in-typescript-maybe-monad/) and [part 2](https://codewithstyle.info/advanced-functional-programming-typescript-monads-generators/), let's move on to another useful example of monads. In this article, we'll introduce a new type called `Result` which is the functional programming's answer to exceptions. You can find the source code for this article [here](https://github.com/miloszpp/typescript-monads/tree/part-3-exceptions-and-result/src). 

Exceptions
----------

Exceptions are a very popular way of handling errors and unexpected situations in code. They are present in mainstream languages such as Java and C# and of course JavaScript. Interestingly, some new programming languages (such as Rust) deliberately didn't introduce exceptions. Are exceptions _compatible_ with functional programming? Unfortunately, not so much. For example, pure functions shouldn't have side effects. Throwing an exception is actually kind of side effect - it can lead to termination of the program. Worse than that, exceptions introduce some unpredictability to the code.

```typescript
    function divide(x: number, y: number): number {
        if (y === 0) 
            throw new Error('Cannot divide by zero');
        return x / y;
    }
```

Although the type signature tells us that `divide` returns a `number`, this is not always the case. We have to be very careful and make sure that we remember to handle the error. However, there is nothing in the type system that will make sure that we don't forget to do that.

Better exceptions
-----------------

How can we make it explicit that something can go wrong inside `divide`? Let's create a new type `Result<TSuccess, TFailure>`. Remember how `Maybe<T>` could either be `Some` or `None`? Similarly, `Result<TSuccess, TFailure>` can either be `Success` which represents the happy path or `Failure` which means that something went wrong and `TFailure` is the type of the error. In other words, instances of our new type can either contain a valid result or an information about what went wrong. In contrast to exceptions, it is now explicit that an error can happen. What's more, we know exactly what kinds of errors we have to deal with (`TFailure` tells us so).

Implementing `Result`
---------------------

Let's start with the following definition.

```typescript
    export class Result<TSuccess, TFailure> {
        private constructor(
            private value: TSuccess,
            private errorValue: TFailure
        ) {}
    
        static success<TSuccess, TFailure>(value: TSuccess) {
            return new Result<TSuccess, TFailure>(value, null);
        }
    
        static failure<TSuccess, TFailure>(errorValue: TFailure) {
            return new Result<TSuccess, TFailure>(null, errorValue);
        }
    }
```

We've created a class that can only be constructed in two ways - via `success` or `failure` static methods. The class internally stores either a `value` representing valid result or `errorValue` containing information about what went wrong. Let's start with a simple method that extracts a value from `Result`. Remember that an instance of `Result` can either be a `Success` or a `Failure`. Therefore, when extracting the value we always have to assume that an error could have occurred. We need to provide `handleError` function which can deal with this error.

```typescript
    get(handleError: (errorValue: TFailure) => TSuccess): TSuccess {
        if (this.value === null) {
            return handleError(this.errorValue);
        } else {
            return this.value;
        }
    }
```

Similarly to `Maybe`, we need some operations to be able to conveniently work with `Result` types. Let's start with `map`. In the _happy_ scenario, it will take a function that will be applied to the value stored inside `Result`. However, if `Result` contains an error, it will simply ignore the provided function and return a `failure`.

```typescript
    map<R>(f: (wrapped: TSuccess) => R): Result<R, TFailure> {
        if (this.value === null) {
            return Result.failure<R, TFailure>(this.errorValue);
        } else {
            return Result.success<R, TFailure>(f(this.value));
        }
    }
```

However, it might be the case that the operation that we want to perform on the value stored inside `Result` returns a `Result` itself! In such case, we need `flatMap`.

```typescript
    flatMap<R>(f: (wrapped: TSuccess) => Result<R, TFailure>): Result<R, TFailure> {
        if (this.value === null) {
            return Result.failure<R, TFailure>(this.errorValue);
        } else {
            return f(this.value);
        }
    }
```

`Result` in practice
--------------------

Great, we're now ready to put our new type to work. Let's adjust the code from the previous posts so that instead of using `Maybe` to represent potentially empty result, it uses `Result` to represent the potentially failed result.

```typescript
    findById(id: number): Result<Employee, string> {
        const results = this.employees.filter(employee => employee.id === id);
        return results.length 
            ? Result.success(results[0]) 
            : Result.failure("Employee does not exist");
    }
```

We've updated the `findById` method so that it wraps the returned employee inside `Result.success`, provided that it was available. Otherwise, it returns `Result.failure` with an error message describing what went wrong. Therefore, `TFailure` will be a `string` in our case. Next, let's update the model. Now `Employee.supervisorId` is a `Result` as well! We treat a situation when an employee does not have a supervisor as kind of an _error_.

```typescript
    export interface Employee {
        id: number;
        name: string;
        supervisorId: Result<number, string>;
    }
    
    private employees: Employee[] = [
        { id: 1, name: "John", supervisorId: Result.failure("No supervisor") },
        { id: 2, name: "Jane", supervisorId: Result.success(1) },
        { id: 3, name: "Joe", supervisorId: Result.success(2) },
    ];
```

Now we need to make some adjustments to the usages of the above code inside `main.ts` file. Firstly, let's change the event listener code to create a `Result` instance based on the content of the HTML input. Next, the `Result` is passed to `getSupervisorName` function which will return a `Result` as well (as we will see in a moment). Finally, when extracting the value from the `Result` instance, we provide a callback to handle the potential error.

```typescript
    findEmployeeButtonEl.addEventListener("click", () => {
        const inputResult: Result<string, string> = employeeIdInputEl.value 
            ? Result.success(employeeIdInputEl.value)
            : Result.failure("No employee id provided");
        const supervisorNameOrError = getSupervisorName(inputResult)
            .get(error => `Error occured: ${error}`);
        searchResultsEl.innerText = `Supervisor name: ${supervisorNameOrError}`;
    });
```

Finally, the `getSupervisorName` function. And this is the most interesting part of the article because... **the function looks almost exactly the same as in the case of `Maybe`**!

```typescript
    function getSupervisorName(enteredIdResult: Result<string, string>): Result<string, string> {
        return enteredIdResult
            .flatMap(safeParseInt)
            .flatMap(employeeId => repository.findById(employeeId))
            .flatMap(employee => employee.supervisorId)
            .flatMap(supervisorId => repository.findById(supervisorId))
            .map(supervisor => supervisor.name);
    }
    
    function safeParseInt(numberString: string): Result<number, string> {
        const result = parseInt(numberString);
        return isNaN(result)
            ? Result.failure("Invalid number format") : Result.success(result);
    }
```

The only adjustments are type signatures and the `safeParseInt` function. It turns out that `map` and `flatMap` operations are so generic that they can handle two distinct scenarios with the same piece code. I hope you can see the power of monads now! You can now run the program and enjoy nice error messages. Try out different scenarios such as providing non-existent id, id of an employee without a supervisor, non-numeric id, etc.

Summary
-------

In this article, we saw how to use monads to replace exceptions with a more functional-friendly approach. Thanks to the `Result` type, we can make it explicit that a function can fail. What's more, we force the caller to always assume that something could have gone wrong and provide an error handler. 

Error handling in this example is rather simplified. We use strings to convey the error message. However, there is nothing stopping you from using more advanced types in order to pass more meaningful information. For example, you could use [discriminated unions](https://codewithstyle.info/typescript-discriminated-union-types/) to represent different kinds of errors. What do you think about this approach to error handling? Do you think it's more readable than traditional exceptions? **Share your thoughts in comments**!
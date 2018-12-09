---
title: 'Advanced functional programming in TypeScript: Maybe monad'
url: 616.html
id: 616
categories:
  - JavaScript
  - TypeScript
date: 2018-02-19 08:00:53
tags:
  - functional programming
  - monads
  - typescript
---

With this post, I would like to start a short series about **monads**. If you are familiar with some functional programming techniques in JavaScript (such as immutability or pure functions), this is a great next step to go deeper into this amazing paradigm. Regardless of whether you've never heard about monads or have heard about them but never really understood them, this series will strive to explain then in simple, practical terms. 

I've already tackled this topic on the blog a few times (see [monads in C#](https://codewithstyle.info/understand-monads-linq/) and [monads in Scala](https://codewithstyle.info/scalas-option-monad-versus-null-conditional-operator-in-c/)) but this time I would like to explore how monads can be useful in the front-end world. One final word - I chose TypeScript over JavaScript because it's just easier to talk about monads in a strongly-typed language. You don't have to be a TypeScript expert to understand the article. 

**You can find all the code from the series in [this repository](https://github.com/miloszpp/typescript-monads). Check the commit history for code relevant to the specific part of the series.** 

Let's get ready for our monadic journey!

Background
----------

We're going to build a simple application that implements the following scenario: _A company has a hierarchical employee structure (each employee can have another employee as a supervisor). As a user, I would like to be able to enter employee ID (a numeric value) and see their supervisor's name._ Let's start with a plain, non-monadic implementation. Here is some HTML for the user interface:

```html
    <body>
        <h1>Find employee's supervisor</h1>
        <p>
            <label for="employeeId">Enter employee ID</label>
            <input type="text" name="employeeId" id="employeeIdInput" />
        </p>
        <p>
            <button type="button" id="findEmployeeButton">Find supervisor's name</button>
        </p>
        <p id="searchResults"></p>
    </body>
```

The HTML consists of an input for the employee's ID and a button to search for the employee's supervisor's name. And here comes the code that orchestrates this form:

```typescript
    import { EmployeeRepository } from "./employee.repository";
    
    const employeeIdInputEl = document.getElementById("employeeIdInput") as HTMLInputElement;
    const findEmployeeButtonEl = document.getElementById("findEmployeeButton");
    const searchResultsEl = document.getElementById("searchResults");
    
    const repository = new EmployeeRepository();
    
    findEmployeeButtonEl.addEventListener("click", () => {
        const supervisorName = getSupervisorName(employeeIdInputEl.value);
        if (supervisorName) {
            searchResultsEl.innerText = `Supervisor name: ${supervisorName}`;
        } else {
            searchResultsEl.innerText = "Could not find supervisor for given id";
        }
    });
    
    function getSupervisorName(enteredId: string) {
        if (enteredId) {
            const employee = repository.findById(parseInt(enteredId));
            if (employee && employee.supervisorId) {
                const supervisor = repository.findById(employee.supervisorId);
                if (supervisor) {
                    return supervisor.name;
                }
            }
        }
    }
```

Firstly, we get hold of some HTML elements. Next, we attach a click handler to the button. Inside the handler, we invoke the `getSupervisorName` function which holds all of the actual logic (we will get back to it soon). Finally, we update the `p` tag with search results. Finally, let's have a quick look at the `EmployeeRepository` class:

```typescript
    import { Employee } from "./employee.model";
    
    export class EmployeeRepository {
        private employees: Employee[] = [
            { id: 1, name: "John" },
            { id: 2, name: "Jane", supervisorId: 1 },
            { id: 3, name: "Joe", supervisorId: 2 },
        ];
    
        findById(id: number) {
            const results = this.employees.filter(employee => employee.id === id);
            return results.length ? results[0] : null;
        }
    }
```

It's just an in-memory storage of the employee hierarchy with some hardcoded values. The `Employee` interface could look like this:

```typescript
    export interface Employee {
        id: number;
        name: string;
        supervisorId?: number;
    }
```

Nesting, nesting, nesting
-------------------------

As promised, let's focus on the `getSupervisorName` function.

```typescript
    function getSupervisorName(enteredId: string) {
        if (enteredId) {
            const employee = repository.findById(parseInt(enteredId));
            if (employee && employee.supervisorId) {
                const supervisor = repository.findById(employee.supervisorId);
                if (supervisor) {
                    return supervisor.name;
                }
            }
        }
    }
```

As we can see, the function body involves several levels of nesting. This is because many things can go wrong during the search for the supervisor.

*   the user can click the button without typing anything in the ID field
*   there can be no employee for given ID
*   the employee we're looking for can have no supervisor (e.g. they're a CEO or an independent consultant)
*   there can be no employee with ID equal to the employee's supervisor's ID (inconsistency in the hierarchy)

In other words, there are many operations involved and each of them can return an **empty result** (e.g. **empty** input field, **empty** search result, etc.). The function needs to handle all of these edge cases and hence the deep nesting of `if` statements. Is there anything wrong with it? I think yes:

*   when writing such code, it's easy to miss some of the edge cases and the compiler won't stop us from doing so
*   such code is not very readable

Let's see how to solve both of these problems.

Introducing `Maybe`
-------------------

One way of simplifying code is to identify a pattern and create an abstraction that hides the implementation details of this pattern. The recurring theme in the `getSupervisorName` function is the nesting of `if` statements.

```typescript
    if (result) {
      const nextResult = operation(result);
      if (nextResult) {
         // and so on...
      }
    } // else stop
```

But how to create an abstraction over such a pattern? The reason we have to do these `if` checks is that the value stored inside `result` can be empty. We'll create a simple wrapper type that holds a simple value and is aware of whether the value is empty (ie. `null` or `undefined` or empty string) or not. Let's call this wrapper type `Maybe`.

```typescript
    export class Maybe<T> {
        private constructor(private value: T | null) {}
    
        static some<T>(value: T) {
            if (!value) {
                throw Error("Provided value must not be empty");
            }
            return new Maybe(value);
        }
    
        static none<T>() {
            return new Maybe<T>(null);
        }
    
        static fromValue<T>(value: T) {
            return value ? Maybe.some(value) : Maybe.none<T>();
        }
    
        getOrElse(defaultValue: T) {
            return this.value === null ? defaultValue : this.value;
        }
    }
```

Instances of `Maybe` hold a `value` that can either be an actual value or `null`. Here, `null` is the internal representation of an empty value. The constructor is private so you can only create `Maybe` instances by calling `some` or `none` static methods. `fromValue` is a convenience method that transforms a regular value to a `Maybe` instance. Finally, `getOrElse` is a safe way of extracting the value contained by `Maybe`. The caller has to provide the default value that will be used in case `Maybe` is empty. So far, so good. We can now explicitly say that the result returned by some method can be empty. Let's change the `findById` method on `EmployeeRepository`:

```typescript
    findById(id: number): Maybe<Employee> {
        const results = this.employees.filter(employee => employee.id === id);
        return results.length ? Maybe.some(results[0]) : Maybe.none();
    }
```

Note that the return type of `findById` is now more meaningful and better captures the programmer's intention. `findById` can indeed return an empty value if an employee with given ID doesn't exist inside the repository. What's more, we can change the `Employee` interface to explicitly state the fact that `supervisorId` can be empty:

```typescript
    export interface Employee {
        id: number;
        name: string;
        supervisorId: Maybe<number>;
    }
```

We'll now add some operations to make `Maybe` type more useful. You know the `map` method that you can call on arrays, right? It applies a given function to every element of an array. If we look at `Maybe` as at a special array that can have from zero to one elements, it turns out that defining `map` on it totally makes sense.

```typescript
    map<R>(f: (wrapped: T) => R): Maybe<R> {
        if (this.value === null) {
            return Maybe.none<R>();
        } else {
            return Maybe.some(f(this.value));
        }
    }
```

Our `map` takes a function `f` that transforms the element wrapped by `Maybe` and returns a new `Maybe` with the result of the transformation. If `Maybe` was a `none` then the result of `map` will also be an empty `Maybe` (just like calling `map` on an empty array would give you an empty array). `R` is the type parameter representing the type returned by `f` transformation. But how is this `map` useful? The original version of the `getSupervisorName` function included the below `if` statement:

```typescript
    const supervisor = repository.findById(employee.supervisorId);
    if (supervisor) {
        return supervisor.name;
    }
```

But `findById` returns a `Maybe` now! And we have the `map` operation available which, accidentally, has exactly the same semantics as the `if` statement above! Therefore, we can rewrite the above piece like this:

```typescript
    const supervisor: Maybe<Employee> = repository.findById(employee.supervisorId);
    return supervisor.map(s => s.name);
```

Didn't we just hide the `if` statement behind an abstraction? Yes, we did! However, we're not ready to rewrite the whole function in such style yet.

Maybe `map`, or maybe `flatMap`?
--------------------------------

Using `map` works fine for transformations such as above. But how about this one?

```typescript
    const employee = repository.findById(parseInt(enteredId));
    if (employee && employee.supervisorId) {
        const supervisor = repository.findById(employee.supervisorId);
        // ...
    }
```

We could try to rewrite it using `map`:

```typescript
    const employee: Maybe<Employee> = repository.findById(parseInt(enteredId));
    const supervisor: Maybe<Maybe<Employee>> = employee.map(e => repository.findById(e.supervisorId));
```

See the problem? The type of `supervisor` is `Maybe<Maybe<Employee>>`. This is because our transformation function now maps from a regular value to a `Maybe` (and previously it was mapping from regular value to a regular value). Is there a way to transform `Maybe<Maybe<Employee>>` to a simple `Maybe<Employee>`? In other words, we would like to **flatten** our `Maybe`. Again, there is an analogy to arrays. You can flatten nested array `[[1, 2, 3], [4, 5, 6]]` to `[1, 2, 3, 4, 5, 6]`. We'll add a new operation to `Maybe` and call it `flatMap`. It's just like `map` but it also flattens the result so that we don't end up with nested `Maybe`s.

```typescript
    flatMap<R>(f: (wrapped: T) => Maybe<R>): Maybe<R> {
        if (this.value === null) {
            return Maybe.none<R>();
        } else {
            return f(this.value);
        }
    }
```

The implementation is pretty simple. If given instance of `Maybe` is not empty then we extract the wrapped value, apply the provided function and simply return the result (which can either be empty or not empty). If the instance was empty, we simply return an empty `Maybe`. Note the type signature of `f`. Previously, it was mapping from `T` to `R`. Now, it's mapping from `T` to `Maybe<R>`. Thanks to the addition of `flatMap`, we can now rewrite the above code piece like this:

```typescript
    const employee: Maybe<Employee> = repository.findById(parseInt(enteredId));
    const supervisor: Maybe<Employee> = employee.flatMap(e => repository.findById(e.supervisorId));
```

Maybe Monad in action
---------------------

Now, we've got all we need to rewrite the `getSupervisorName` function.

```typescript
    function getSupervisorName(maybeEnteredId: Maybe<string>): Maybe<string> {
        return maybeEnteredId
            .flatMap(employeeIdString => Maybe.fromValue(parseInt(employeeIdString))) // parseInt can fail
            .flatMap(employeeId => repository.findById(employeeId))
            .flatMap(employee => employee.supervisorId)
            .flatMap(supervisorId => repository.findById(supervisorId))
            .map(supervisor => supervisor.name);
    }
```

We've eliminated all of the nested `if` statements! The `getSupervisorName` function's body is now an elegant pipeline of transformations applied to the input value. We've hidden the details of handling empty results because they're just boilerplate and obfuscate the real intention of the code. They're now taken care of by `Maybe`. Note that if any of the operations inside `flatMap` returned a `none`, it would cause the whole thing to immediately return a `none`. This is actually the same behaviour that we had with nested `if` statements. Here is an example of how the function can be used inside the click handler:

```typescript
    findEmployeeButtonEl.addEventListener("click", () => {
        const supervisorName = getSupervisorName(Maybe.fromValue(employeeIdInputEl.value));
        searchResultsEl.innerText = `Supervisor name: ${supervisorName.getOrElse("could not find")}`;
    });
```

And, guess what, `Maybe` is a **monad**! The formal definition of a monad is that it's a container type that has two operations:

*   `return` \- which creates an instance of the type from a regular value (`some` and `none` in our case)
*   `bind` \- which lets you combine monadic values (`flatMap` in our case)

There are also some monadic laws that every monad has to follow but let's not dive into it yet. For now, you have to trust me that our `Maybe` implementation follows these laws :)

Summary
-------

In this first post of the series, we've created our first monad. The purpose of the `Maybe` monad is to abstract away handling of empty values. Thanks to the introduction of this type, we can now write code without having to worry about empty results. In the next article, we'll see how thanks to TypeScript we can write code that uses monads in an even more readable way. Do you find monads interesting? Do you feel like you understand the concept now or is it still a mystery? **Please let me know in comments!**
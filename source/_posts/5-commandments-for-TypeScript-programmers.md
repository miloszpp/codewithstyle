---
title: 5 Commandments for TypeScript programmers
date: 2019-07-16 20:09:53
tags:
    - typescript
    - best practices
image: /images/posts/clouds.jpg
---

More and more projects and teams are adopting TypeScript. However, there is a massive difference between just using TypeScript and taking the most out of it. 

I present to you this list of high-level TypeScript best practices that will help you take advantage of TypeScript to the fullest possible extent.

## Do not lie

**Types are a contract.** What does it mean? When you implement a function, its type is a promise to other developers (or to your future self!) that when they call it, it will return a specific kind of value.

In the following example, the type of `getUser` promises that it will return an object that will **always** have two properties: `name` and `age`.

```typescript
interface User {
  name: string;
  age: number;
}

function getUser(id: number): User { /* ... */ }
```

TypeScript is a very flexible language. It's full of compromises made in order to make its adoption easier. For example, it allows you to implement `getUser` like this:

```typescript
function getUser(id: number): User {
  return { age: 12 } as User;
}
```

Don't do this! It's a LIE. By doing this, you LIE to other developers (who will use this function in their functions). They expect the object returned by `getUser` to always have some `name`. But it doesn't! Then, what happens when your teammate writes `getUser(1).name.toString()`? You know it well...

Of course, this lie seems very obvious. However, when working with a huge codebase, you will often find yourself in a situation when a value you want to return (or pass) _almost_ matches the expected type. **Figuring out the reason for type mismatch takes time and effort** and you are in a hurry... so you decide to cast. 

However, by doing this, you **violate the holy contract**. It's ALWAYS better to take time to figure out why types do not match than to do the cast. It's very likely that some runtime bug is lurking under the surface.

**Don't lie. Respect your contracts.**

## Be precise

**Types are documentation.** When you document a function, don't you want to convey as much information as possible?

```javascript
// Returns an object
function getUser(id) { /* ... */ }

// Returns an object with two properies: name and age
function getUser(id) { /* ... */ }

// If id is a number and a user with given id exists,
// returns an object with two properies: name and age.
// Otherwise, returns undefined.
function getUser(id) { /* ... */ }
```

Which comment for `getUser` would you prefer? The more you know about what it returns, the better. For example, knowing that it could return `undefined`, you can write an `if` statement to check if the value it returned is defined before accessing its properties.

It's exactly the same with types. The more precise a type is, the more information it conveys. 

```typescript
function getUserType(id: number): string { /* ... */ }

function getUserType(id: number): 'standard' | 'premium' | 'admin' { /* ... */ }
```

The second version of `getUserType` is much more informative, and hence it puts the caller in a much better situation. It's easier to handle a value if you know that it is **for sure** (contracts, remember?) one of the three strings than if it can be any string. For starters, you know for sure that the value is not an empty string.

Let's see a more realistic example. `State` type represents the state of a component that fetches some data from the backend. Is this type precise?

```typescript
interface State {
  isLoading: boolean;
  data?: string[];
  errorMessage?: string;
}
```

The consumer of this type must handle some unlikely combinations of property values. For example, it's not possible that both `data` and `errorMessage` will be defined (data fetching can either be successful or result in an error). 

We can make the type much more precise with the help of discriminated union types:

```typescript
type State =
   | { status: 'loading' }
   | { status: 'successful', data: string[] }
   | { status: 'failed', errorMessage: string };
```

Now, the consumer of this type has much more information. They don't need to handle illegal combinations of property values.

**Be precise. Convey as much information as possible in your types.**

## Start with types

Since types are both contract and documentation, they're great for **designing** your functions (or methods).

There are many articles around in the internet that advise software engineers to **think before they write code**. I totally agree with this approach. It's tempting to jump straight into code, but it often leads to some bad decisions. Spending some time thinking about the implementation always pays off.

Types are super helpful in this process. _Thinking_ can result in writing down the type signatures of functions involved in your solution. It's awesome because it lets you focus on _what_ your functions do instead of _how_ they achive it.

React JS has a concept of Higher Order Components. They are functions that augment given component in some way. For example, you could create a `withLoadingIndicator` Higher Order Component that adds a loading indicator to an existing component.

Let's write the type signature for this function. It takes a component and returns a component. We can use React's `ComponentType` to indicate a component.

`ComponentType` is a generic type parameterized by the type of properties of the component. `withLoadingIndicator` takes a component and returns a new component that either shows the original component or shows a loading indicator. The decision is made based on the value of a new boolean property `isLoading`. Therefore, the resulting component should require the same properties as the original component plus the new property.

Let's finalize the type. `withLoadingIndicator` takes a component of type `ComponentType<P>` where `P` denotes the type of the properties. It returns a component with augmented properties of type `P & { isLoading: boolean }`.

```typescript
const withLoadingIndicator = <P>(Component: ComponentType<P>) 
    : ComponentType<P & { isLoading: boolean }> =>
        ({ isLoading, ...props }) => { /* ... */ }
```

Figuring out the type of this function forced us to think about its input and its output. In other words, it made us _design it_. Writing the implementation is a piece of cake now.

**Start with types. Let types force you to design before implementing.**

## Embrace strictness

The first three points require you to pay a lot of attention to types. Fortunately, you are not alone in the task - TypeScript compiler will often let you know when your types lie or when they're not precise enough.

You can make the compiler even more helpful by enabling `--strict` compiler flag. It is a meta flag that enables all strict type checking options: `--noImplicitAny`, `--noImplicitThis`, `--alwaysStrict`, `--strictBindCallApply`, `--strictNullChecks`, `--strictFunctionTypes` and `--strictPropertyInitialization`.

What do they do? In general, enabling them results in more TypeScript compiler errors. This is good! More compiler errors mean more help from the compiler.

Let's see how enabling `--strictNullChecks` helps you identify a lie.

```typescript
function getUser(id: number): User {
    if (id >= 0) {
        return { name: 'John', age: 12 };
    } else {
        return undefined;
    }
}
```

The type of `getUser` promises that it will always return a `User`. However, as you can see in the implementation, it can also return an `undefined` value!

Fortunately, enabling  `--strictNullChecks` results in a compiler error:

```
Type 'undefined' is not assignable to type 'User'.
```

TypeScript compiler detected the lie. You can get rid of the error by telling the truth: 

```typescript
function getUser(id: number): User | undefined { /* ... */ }
```

**Embrace type checking strictness. Let the compiler watch your steps.**

## Stay up to date

TypeScript language is being developed at a very fast pace. There is a new release every two months. Each release brings in significant language improvements and new features.

It is often the case that new language features allow for more precise types and stricter type checking.

For example, version 2.0 introduced Discriminated Union Types (which I mentioned in _Be precise_).

Version 3.2 introduced `--strictBindCallApply` compiler option which enables correct typing of `bind`, `call` and `apply` functions.

[Version 3.4 improved type inference in higher order functions](/TypeScript-3-4-hidden-gem-propagated-generic-type-arguments/), making it easier to use precise type when writing code in functional style.

My point here is that it really pays off to be familiar with language features introduced in the latest releases of TypeScript. They can often help you adhere to the other four commandments from this list.

A good starting point is [the official TypeScript roadmap](https://github.com/Microsoft/TypeScript/wiki/Roadmap). It's also a good idea to check out [TypeScript section of the Microsoft Devblog](https://devblogs.microsoft.com/typescript/) regularly as all release announcements are made there.

**Stay up to date with new language features and make the work for you.**

## Summary

I hope you find the list useful. Like anything in life, these commandments shouldn't be followed blindly. However, I firmly believe those rules will make you a better TypeScript programmer.

I'd love to hear your thoughts about it in the comments section.
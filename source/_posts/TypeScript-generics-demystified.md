---
title: "The ultimate explanation of TypeScript generics: functions"
date: 2019-10-15 22:15:34
tags:
  - typescript
  - generics
  - basics
image: /images/posts/gears.jpg
---

Recently I surveyed the readers of this blog to find out what TypeScript features people find difficult to understand. Generics were mentioned quite often. In this article, I'm going to equip you with a mental model that will let you understand **generic functions** properly (I'll focus on **generic types** in another article).

The concept of generics is not a very new one - it has been present in different programming languages (such as Java, C# or C++) for a long time. However, for folks without background in a statically typed language, generics might appear complicated. Therefore, I'm not going to make any assumptions and will explain generics completely from scratch.

## Motivation

Let's say you are adding types to some JavaScript codebase and you encounter this function:

```javascript
function getNames(persons) {
  const results = [];
  for (let person of persons) {
    results.push(person.name);
  }
  return results;
}
```

Typing this function is straightforward. It accepts an array of person objects as a parameter and returns an array of names (strings). For the person object, you can either create a `Person` interface or use one that you've already created.

```typescript
interface Person {
  name: string;
  age: number;
}

function getNames(persons: Person[]): string[] {
  /* ... */
}
```

Next, you notice that you don't actually need this function. Instead, you can use the built-in `Array.map` method.

```typescript
const persons: Person[] = [
  /* ... */
];
const names = persons.map(person => person.name);
```

Hmm, but what about types? You check the type of `names` and realize that it has been correctly inferred to `string[]`! How does TypeScript achieve such an effect?

To properly understand this, let's try to type the following implementation of `map` function.

```javascript
function map(items, mappingFunction) {
  const results = [];
  for (let item of items) {
    results.push(mappingFunction(item));
  }
  return results;
}

const names = map(persons, person => person.name);
```

The main issue with typing `map` is that you don't know anything about the type of the elements of the array it will be called with. What makes `map` so cool is that it works with _any_ kind of array!

```javascript
// Works with array of Persons
const names = map(persons, person => person.name);
// Works with array of names too
const uppercaseNames = map(names, name => name.toUpperCase());
// Works even with an array of numbers!
const evenNumbers = map([1, 2, 3, 4, 5], n => n * 2);
```

## Let's use `any`!

As a first step, let's try using `any` type to `map` this function.

```typescript
function map(items: any[], mappingFunction: (item: any) => any): any[] {
  /* ... */
}
```

Let's break this down. `map` has two parameters. The type of the first one (`items`) is `any[]`. We tell the type system that we want `items` to be an array, but we don't care about the type of those items. The type of the second parameter (`mappingFunction`) is a function that takes `any` and returns `any`. Finally, the return type is again `any[]` - an array of _anything_.

Did we gain anything by doing this? Sure! TypeScript now won't allow us to call `map` with some nonsensical arguments:

```typescript
// 🔴 Error: 'hello' is not an array
map("hello", (person: Person) => person.name);
// 🔴 Error: 1000 is not a function
map(persons, 1000);
```

Unfortunately, the types we provided are not precise enough. The purpose of TypeScript is to catch possible runtime errors earlier, at compile-time. However, the following calls won't give any compile errors.

```typescript
// The second argument is a function that only works on numbers, not on `Person` objects.
// This would result in a runtime error.
map(persons, n => n + 5);
// We tell TypeScript that `numbers` is an array of strings while in fact it will be an array of numbers.
// The second line results in a runtime error.
const names: string[] = map(persons, person => person.age);
names[0].toLowerCase();
```

How can we improve the typing of `map` so that above examples would result in a compile-time error? Enter generics.

## Generic functions

Generic function is (in this case) a way of saying "this function works with any kind of array" and maintaining type safety at the same time.

```typescript
function map<TElement, TResult>(
  items: TElement[],
  mappingFunction: (item: TElement) => TResult
): TResult[] {
  /* ... */
}
```

We replaced `any` with `TElement` and `TResult` type parameters. Type parameters are like _named `any`s_. Typing `items` as `TElement[]` still means that it is an array of anything. However, because it's _named_, it lets us establish relationships between types of function parameters and the return type.

Here, we've just expressed the following relationships:

- `mappingFunction` takes anything as a parameter, but it must be _the same type of "anything"_ as the type of elements of `items` array
- `mappingFunction` can return anything, but whatever type it returns, it will be used as the type of elements of the array returned by `map` function

The picture below demonstrates these relationships. Shapes of the same color have to be of the same type.

![Generic `map`](/images/posts/generic-functions.png)

You might have noticed the `<TElement, TResult>` thing that we added next to `map`. Type parameters have to be declared explicitly using this notation. Otherwise, TypeScript wouldn't know if `TElement` is a type argument or an actual type.

BTW, for some reason, it is a common convention to use single-character names for type parameters (with a strong preference for `T`). I'd strongly recommend using full names, especially when you are not that experienced with generics. On the other hand, it's a good idea to prefix type arguments with `T`, so that they're easily distinguishable from regular types.

## Calling generic functions

How to call a generic function? As we saw, generic functions have type parameters. These parameters are replaced with actual types "when" the function is called (technically, it's all happening at compile-time). You can provide the actual types using angle brackets notation.

```typescript
map<Person, string>(persons, person => person.name);
```

Imagine that by providing type arguments `TElement` and `TResult` become replaced with `Person` and `string`.

![Generic `map`](/images/posts/generic-functions-2.png)

```typescript
function map<TElement, TResult>(
  items: TElement[],
  mappingFunction: (item: TElement) => TResult
): TResult[] {
  /* ... */
}

// ...becomes...

function map(
  items: Person[],
  mappingFunction: (item: Person) => string
): string[] {
  /* ... */
}
```

Having to provide type arguments, when calling generic functions would be cumbersome. Fortunately, TypeScript can infer them by looking at the types of the arguments passed to the function. Therefore, we end up with the following code.

```typescript
const names = map(persons, person => person.name);
```

Whoohoo! It looks exactly as the JavaScript version, except it's type-safe! Contrary to the first version of `map`, the type of `names` is `string[]` instead of `any[]`. What's more, TypeScript is now capable of throwing a compile error for the following call.

```typescript
// 🔴 Error! Operator '+' cannot be applied to Person and 5.
map(persons, n => n + 5);
```

Here is a very simplified sequence of steps that leads the compiler to throw an error.

1. Compiler looks at the type of `persons`. It sees `Person[]`.
1. According to the definition of `map`, the type of the first parameter is `TElement[]`. Compiler deduces that `TElement` is `Person`.
1. Compiler looks at the second parameter. It should be a function from `Person` to `TResult`. It doesn't know what `TResult` is yet.
1. It checks the body of the function provided as the second argument. It infers that the type of `n` is `Person`.
1. It sees that you're trying to add `5` to `n`, which is of type `Person`. This doesn't make sense, so it throws an error.

## When to use generic functions?

The good news is that, most likely, you will not be creating generic functions very often. It's much more common to call generic functions then to define them. However, it's still very useful to know how generic functions work, as it can help you better understand compiler errors.

As exemplified by `map`, functions that take arrays as parameters are often generic functions. If you look at the typings for `lodash` library, you will see that nearly all of them are typed as generic functions. Such functions are only interested in the fact that the argument is an array, they don't care about the type of its elements.

In React framework, Higher Order Components are generic functions, as they only care about the argument being a component. The type of the component's properties is not important.

In RxJs, most operators are generic functions. They care about the input being and `Observable`, but they're not interested in the type of values being emitted by the observable.

## Summary

Wrapping up:

- generic functions let you achieve type safety for functions that work with many different types of inputs;
- type arguments are very much like `any` type, except they can be used to express relationships between function parameters and the return type;
- calling a generic function is very straightforward thanks to type inference.

I hope this article helped you finally understand generic functions. If not, please let me know!

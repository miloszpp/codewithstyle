---
title: TypeScript conditional types real-life example
date: 2019-02-19 22:42:24
icon: fas fa-code
tags:
  - typescript
  - advanced types
---

I love the [Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html) section of TypeScript docs. There are some amazing features out there. However, the first time I saw some of them, I didn't immediately see what could they be used for. In this article, I'd like to show you a real-world example of using **conditional types**.

You can play with the code [here](https://stackblitz.com/edit/conditional-types).

## Extracting React component's properties

I love using React together with TypeScript. Being able to safely (type-wise) pass properties in JSX is a big win to me. However, once you want to do something non-standard, typing your code properly becomes less obvious.

Some time ago I was implementing a generic function that took a React component as a parameter and returned a type that was based on the type of the component's `props`. I needed a mechanism that would _extract_ the type of `props` from the component's type. It turned out that this can be achieved with a direct application of TypeScript's conditional types.

## Conditional types

Conditional types bring conditional logic to the world of types. They allow you to define something like a _function on types_ that takes a type as an input and based on some condition returns another type.

```typescript
type IsString<T> = T extends string ? "yes" : "no";
```

In this example `IsString` takes `T` and examines it. If `T` is a string then the result would be a `"yes"` literal type (a type whose only possible value is `"yes"` string). Otherwise, it will be a `"no"`.

```typescript
type A = IsString<number>;
const a: A = "hello";
// Type '"hello"' is not assignable to type '"no"'.
```

Look how similiar this is to a regular TypeScript function:

```typescript
const isString = (x: any) => typeof x === "string" ? "yes" : "no";
```

The difference is that a conditional type operates in the world of types while a regular function operates in the world of values.

## Back to React

How can we take advantage of this mechanism and use it to extract the type of React component properties? Let's create a conditional type that checks whether a given type is a React component. 

```typescript
type IsReactComponent<T> =
  T extends React.Component<any> ? "yes" : "no";
```

We had to specify the type parameter for `React.Component` so we used `any`. However, TypeScript lets us do something cooler - we can use the `infer` keyword instead.

```typescript
type IsReactComponent<T> =
  T extends `React.Component`<infer P> ? "yes" : "no";
```

`infer` creates a new type variable `P` that will store the type parameter of `T` if it indeed extends `React.Component`. In our case, it will be exactly what we're looking for - the type of `props`! So, instead of returning `"yes"` literal type, let's simply return `P`.

```typescript
type IsReactComponent<T> =
  T extends React.Component<infer P> ? P : "no";
```

Now, we can assume that this type will only be used with actual React components. We'd like the compilation to fail otherwise. Instead of returning `"no"`, let's return the `never` type. It's a special type that is intended exactly for such situations. We return `never` when we don't want something to happen. If a variable has `never` type then nothing can be assigned to it.

```typescript
type PropsType<C> =
  C extends React.Component<infer P> ? P : never;

class Article extends React.Component<{ content: string }> {
  render = () => <p>{this.props.content}</p>;
}

type ArticleProps = PropsType<Article>;

// type ArticleProps = {
//   content: string;
// }
```

And that's it!

## What about Functional Components?

But hey, React is all about functional components now, isn't it? Let's adjust `PropsType` to take this into account.

```typescript
type PropsType<C> =
  C extends React.Component<infer P> ? P : 
  C extends React.FunctionComponent<infer P> ? P :
  never;
```

A piece of cake! Conditions in conditional types can be nested, just like a regular ternary operator. Also, note that `extends` is not just about classes inheriting from other classes. The condition checks **if a type is assignable from another type**.

```typescript
const Header: React.FunctionComponent<{ text: string }>
  = (props) => <h1>{props.text}</h1>;

type HeaderProps = PropsType<typeof Header>;
```

## Summary

I hope this article convinced you of the usefulness of conditional types in TypeScript. This is just one of many applications of advanced types, so don't hesitate to dive deep into this section of TypeScript docs!
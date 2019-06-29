---
title: The meaning of union and intersection types
date: 2019-06-27 22:44:02
tags:
    - typescript
    - basics
image: /images/posts/intersection.jpg
---

Union types are fairly popular in TypeScript. You might have already used them multiple times. Intersection types are slightly less common. They seem to cause a little bit more confusion.

Did you ever wonder where do those names come from? While you might have some intuition about what a union of two types is, the intersection is usually not understood well.

After reading this article, you will have a better understanding of those types which will make you more confident when using them in your codebases.

## Simple union types

Union type is very often used with either `null` or `undefined`.

```typescript
const sayHello = (name: string | undefined) => { /* ... */ };
```

For example, the type of `name` here is `string | undefined` which means that either a `string` OR an `undefined` value can be passed to `sayHello`.

```typescript
sayHello("milosz");
sayHello(undefined);
```

Looking at the example, you can intuit that a union of types `A` and `B` is a type that accepts both `A` and `B` values.

## Union and intersection of object types

This intuition also works for complex types.

```typescript
interface Foo {
    foo: string;
    xyz: string;
}

interface Bar {
    bar: string;
    xyz: string;
}

const sayHello = (obj: Foo | Bar) => { /* ... */ };

sayHello({ foo: "foo", xyz: "xyz" });
sayHello({ bar: "bar", xyz: "xyz" });
```

`Foo | Bar` is a type that has either all required properties of `Foo` OR all required properties of `Bar`. Inside `sayHello` it's only possible to access `obj.xyz` because it's the only property that is included in both types.

What about the intersection of `Foo` and `Bar`, though?

```typescript
const sayHello = (obj: Foo & Bar) => { /* ... */ };

sayHello({ foo: "foo", bar: "bar", xyz: "xyz" });
```

Now `sayHello` requires the argument to have both `foo` AND `bar` properties. Inside `sayHello` it's possible to access both `obj.foo`, `obj.bar` and `obj.xyz`.

Hmm, but what does it have to _intersection_? One could argue that since `obj` has properties of both `Foo` and `Bar`, it sounds more like a union of properties, not intersection. Similarly, a union of two object types gives you a type that only has the intersection of properties of constituent types.

It sounds confusing. I even stumbled upon a [GitHub issue](https://github.com/Microsoft/TypeScript/issues/18383) in TypeScript repository ranting about naming of these types. To understand the naming better we need to look at types from a different perspective.

## Set theory

Do you remember a concept called _sets_ from math classes? In mathematics, a set is a collection of objects (for example numbers). For example, `{1, 2, 7}` is a set. All positive numbers can also form a set (an infinite one).

Sets can be added together (a **union**). A union of `{1, 2}` and `{4, 5}` is `{1, 2, 4, 5}`.

Sets can also be intersected. **Intersection** of two sets is a set that only contains those numbers that are present in both sets. So, an intersection of `{1, 2, 3}` and `{3, 4, 5}` is `{3}`.

Let's imagine two sets: `Squares` and `RedThings`.

The union of `Squares` and `RedThings` is a set that contains both squares and red things.

However, the intersection of `Squares` and `RedThings` is a set that only contains **red squares**.

![Intersection of sets](/images/posts/intersection-shapes.png)

## Relationship between types and sets

Computer science and mathematics overlap in many places. One of such places is type systems.

A type, when looked at from a mathematical perspective, is a **set of all possible values of that type**. For example the `string` type is a set of all possible strings: `{'a', 'b', 'ab', ...}`. Of course, it's an infinite set.

Similarly, `number` type is a set of all possible numbers: `{1, 2, 3, 4, ...}`.

Type `undefined` is a set that only contains a single value: `{ undefined }`.

What about object types (such as interfaces)? Type `Foo` is a **set of all object that contain `foo` and `xyz` properties**.

## Understanding union and intersection types

Armed with this knowledge, you're now ready to understand the meaning of union and intersection types.

Union type `A | B` represents a set that is a union of the set of values associated with type `A` and the set of values associated with type `B`.

Intersection type `A & B` represents a set that is an intersection of the set of values associated with type `A` and the set of values associated with type `B`.

Therefore, `Foo | Bar` represents **a union** of the set of objects having `foo` and `xyz` properties and the set of objects having `bar` and `xyz`. Objects belonging to such set all have `xyz` property. Some of them have `foo` property and the others have `bar` property.

`Foo & Bar` represents **an intersection** of the set of objects having `foo` and `xyz` properties and the set of objects having `bar` and `xyz`. In other words, the set contains objects that belong to sets represented by both `Foo` and `Bar`. Only objects that have all three properties (`foo`, `bar` and `xyz`) belong to the intersection.

## Real-world example of intersection type

Union types are quite widespread so let's focus on an example of an intersection type.

In React, when you declare a class component, you can parameterise it with thy type of its properties:

```typescript
class Counter extends Component<CounterProps> { /* ... */ }
```

Inside the class, you can access the properies via `this.props`. However, the type of `this.props` is not simply `CounterProps`, but: 

```typescript
Readonly<CounterProps> & Readonly<{ children?: ReactNode; }>
```

The reason for this is that React components can accept children elements:

```typescript
<Counter><span>Hello</span></Counter>
```

The children element tree is accessible to the component via `children` prop. The type of `this.props` reflects that. It's an intersection of (readonly) `CounterProps` and a (readonly) object type with an optional `children` property. 

In terms of sets, it's an intersecion of the set of objects that have properties as defined in `CounterProps` and the set of objects that have optional `children` property. The result is a set of objects that have both all properties of `CounterProps` and the optional `children` property.

## Summary

That's it! I hope this article helps you wrap your head around union and intersection types. As it's often the case in computer science, understanding the fundamentals makes you better at grasping programming concepts.
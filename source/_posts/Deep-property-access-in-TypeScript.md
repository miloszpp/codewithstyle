---
title: Deep property access in TypeScript
date: 2019-02-26 18:04:28
icon: fas fa-code
image: /images/posts/space-2638126.jpg
tags:
    - typescript
    - advanced types
---

Strict null checking (enabled with `strictNullChecks` compiler flag) is one of the best things that happened to TypeScript. Thanks to this feature you can make your code a lot safer by eliminating a whole class of bugs during compile time.

However, enabling strict null checks comes at a cost. Adding appropriate conditions might make your code more verbose. This is especially painful in case of accessing deeply nested properties.

In this article, you'll see how to take advantage of **mapped types** to deal with nested properties in an elegant, concise way.

Check out the source code with snippets used in this article [here](https://stackblitz.com/edit/deep-properties).

Many thanks to [mgol](https://twitter.com/m_gol) for the inspiration for the idea behind this article.

---

**Update 1:** *Check out an interesting this discussion on the topic in this [reddit thread](https://www.reddit.com/r/typescript/comments/aynx0o/safe_deep_property_access_in_typescript/).*

**Update 2:** *Many thanks to [Useless-Pickles](https://www.reddit.com/user/Useless-Pickles) who pointed out some issues with the initial version of code in this post. Check out [theirs implementation](https://www.reddit.com/r/typescript/comments/aynx0o/safe_deep_property_access_in_typescript/ei65qw6/?context=3) with improved type safety.*


## Deeply nested properties

Imagine you're working with the following interface:

```typescript
interface Customer {
  name: string;
  company?: {
    name: string;
    address?: {
      city: string;
    }
  }
}
```

At some point, you might want to find out the city of the company of given customer. Without `strictNullChecks`, it would be pretty straightforward. 

```typescript
const c: Customer = /* ... */;
const city = c.company.address.city;
```

Of course, this is very unsafe. With strict null checking enabled, TypeScript forces you to ensure that an object is defined before accessing its property. The least verbose way of doing this is to use the `&&` operator.

```typescript
const city = 
    c && 
    c.company && 
    c.company.address && 
    c.company.address.city;
```

This is not bad, but can we do better?

## Let's try `lodash`?

[Lodash](https://lodash.com) library has a nice utility function `get`. It lets you access a deeply nested property in a safe way. Basically, you can specify a path to the property. If any object on the path is undefined, the function will return undefined. Otherwise, it will return the value of the property.

```typescript
import { get } from 'lodash';

const safeCity = get(c, 'company.address.city');
```

This code is pretty neat and concise. However, the problem with this approach is that it's not type-safe. There is nothing stopping you from making a silly typo and then spending hours figuring that out

```typescript
get(c, 'company.addres.city');
```

So, is there a way to make `get` type-safe?

## Introducing index type query operator

Let's write our own version of `get`. In the first iteration `get` will only accept a single level of nesting (ie. it will handle `get(c, 'company')` properly).

```typescript
function get<
  T, 
  P extends keyof NonNullable<T>
>(obj: T | undefined, prop: P): NonNullable<T>[P] | undefined {
  if (obj) {
    return (obj as NonNullable<T>)[prop];
  } else {
    return undefined;
  }
}
```

The function body is pretty straightforward. What's interesting here is the type signature. `get` is a generic function with two type parameters. 

The first one (`T`) is the type of object from which we want to read the property.

The second one (`P`) is a type that is assignable from `keyof NonNullable<T>`. What is `keyof NonNullable<T>`? It returns a type that is a union of literal types corresponding to all property names of `NonNullable<T>`.

For example, `keyof Customer` is equal to `"name" | "company"`.

Literal type is a type that only has a single possible value. In this instance, `prop` would have to be a string that is equal to either `"name"` or `"company"`.

Finally, why do we use `NonNullable<T>` instead of `T`? `T` can be any type, including one that accepts `null` and `undefined` values. We want to access a property of `T`, so first we need to make sure that it is not `null` nor `undefined`. Hence, we wrap it with `NonNullable`.

Thanks to this type signature, the compiler will make sure that we use a correct string when passing the `prop` argument. Indeed, the following code returns a type error.

```typescript
get(c, 'kompany')
```

## Going deeper

This is cool, but how about deeper nesting?

This is going to be tricky. We need a way to say that the type of N-th argument somehow depends on the type of (N-1)-th argument.

In fact, it is not currently possible to do this for an arbitrary number of arguments in TypeScript. It is one of the limitations of its otherwise powerful type system.

Fear not, the hope is not lost yet! We can cheat a little. In practice, how many levels of nesting are you going to need? 3? 4? 10? The number is not big. We can take advantage of this fact and defined a finite number of overloads for `get`.

```typescript
function get<
  T, 
  P1 extends keyof NonNullable<T>
>(obj: T, prop1: P1): NonNullable<T>[P1] | undefined;

function get<
  T, 
  P1 extends keyof NonNullable<T>,
  P2 extends keyof NonNullable<NonNullable<T>[P1]>
>(obj: T, prop1: P1, prop2: P2): NonNullable<NonNullable<T>[P1]>[P2] | undefined;

function get<
  T, 
  P1 extends keyof NonNullable<T>,
  P2 extends keyof NonNullable<NonNullable<T>[P1]>,
  P3 extends keyof NonNullable<NonNullable<NonNullable<T>[P1]>[P2]>
>(obj: T, prop1: P1, prop2: P2, prop3: P3): NonNullable<NonNullable<NonNullable<T>[P1]>[P2]>[P3] | undefined;

// ...and so on...

function get(obj: any, ...props: string[]): any {
  return obj && props.reduce(
    (result, prop) => result == null ? undefined : result[prop],
    obj
  );
}
```

TypeScript lets us provide multiple type signatures for a function that can handle any number of arguments. We define one signature for each level of nesting that we want to support. For given level of nesting N, we need to define a signature that takes the object and N property names. The type of each property name will have to be one of the keys of the previous property. Once you understand the mechanism, it's pretty straightforward to create these overloads.

We now have a nice, type-safe way to access deeply nested properties!

```typescript
get(c, 'company', 'address', 'city')
```

In fact, this technique is widely used in libraries and frameworks. [Here](https://github.com/ReactiveX/rxjs/blob/master/src/internal/util/pipe.ts), you can observe it being used in RxJS.

BTW, this type signature sometimes gives false negatives. If all properties and the source object itself are not nullable, then the result type should not include `undefined`. Check out [this implementation](https://www.reddit.com/r/typescript/comments/aynx0o/safe_deep_property_access_in_typescript/ei65qw6/?context=3) for more details.

## Summary

In this article, you've seen how to solve a common problem of safely accessing deeply nested properties. On the way, you have learned about index types (the `keyof` operator), literal types and the generic technique of typing functions that accept multiple arguments whose types depend on each other.

Please leave a comment if you enjoyed this article!

Cover photo [source](https://pixabay.com/pl/photos/przestrzeń-deep-space-galaktyka-2638126/).
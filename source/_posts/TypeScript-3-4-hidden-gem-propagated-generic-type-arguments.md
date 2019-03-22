---
title: 'TypeScript 3.4 hidden gem: propagated generic type arguments'
date: 2019-03-18 22:18:05
image:
    /images/posts/gem.jpg
tags:
    - typescript
    - functional programming
    - pointfree style
---

Everyone's excited about incremental builds in the upcoming TypeScript version. However, there is another, at least as interesting, feature in this release. Hidden at the very bottom of the [TypeScript 3.4 RC announcement](https://devblogs.microsoft.com/typescript/announcing-typescript-3-4-rc/) lies a humble section called _Propagated generic type arguments_.

In fact, [Anders Hejlsberg](https://en.wikipedia.org/wiki/Anders_Hejlsberg) himself is excited about this feature 😉

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Really excited about this one...<a href="https://t.co/bbTF9bdFO5">https://t.co/bbTF9bdFO5</a></p>&mdash; Anders Hejlsberg (@ahejlsberg) <a href="https://twitter.com/ahejlsberg/status/1102694547262328832?ref_src=twsrc%5Etfw">March 4, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

So, what's the meaning of this enigmatic update in TypeScript?

**Note:** at the moment of writing TypeScript 3.4 is still a Release Candidate. You can install it by running `npm install typescript@next`.

## Example

Let's have a look at the following example. Imagine that you're fetching a collection of objects from some backend service and you need to map this collection to an array of identifiers.

```typescript
interface Person {
  id: string;
  name: string;
  birthYear: number;
}

function getIds(persons: Person[]) {
    return persons.map(person => person.id);
}
```

Next, you decide to generalize `getIds` function so that it works on any collection of objects having the `id` property.

```typescript
function getIds<T extends Record<'id', string>>(elements: T[]) {
  return elements.map(el => el.id);
}
```

Fair enough. However, the code for this simple function is quite verbose. Can we make it more concise?

## Pointfree style

Sure, we can take advantage of a functional programming technique called **pointfree style**. [Ramda](http://ramdajs.com) is a nice library that will let us compose this function from other functions: `map` and `prop`.

```typescript
import * as R from 'ramda';

const getIds = R.map(R.prop('id'));
```

`map` is _partially applied_ with a mapper function `prop` which extracts the `id` property from any object. The result of `getIds` is a function that accepts a collection of object. You can read a more detailed explanation in my article about [pointfree style](https://codewithstyle.info/Better-RxJS-code-with-pointfree-style/).

Sadly, TypeScript (pre 3.4) has bad news for you. The type of `getIds` is infered to `(list: {}) => {}` which is not exactly what you'd expect.

You can explicitly type `map` but it makes the expression really verbose:

```typescript
const getIds = R.map<Record<'id', string>, string>(R.prop('id'));
```

This is where _propagated generic type arguments_ come in. In TypeScript 3.4 the type of `getIds` will correctly infer to `<T>(list: readonly Record<"id", T>[]) => T[]`. Success!

## Propagated generic type arguments

Now that we now what _propagated generic type arguments_ is about, let's decipher the name.

`R.map(R.prop('id'))` is an example of a situation when we pass a generic function as an argument to another generic function. 

Before version 3.4 TypeScript the type of parameters of inner function type was not _propagated_ to the result type of the call.

## Why should I care?

Even if you're not particularly excited about pointfree style programming, bear in mind that some popular libraries that rely on function composition and partial application and will also benefit from this change.

For example, in [RxJS](https://rxjs.dev) it is possible to compose new operators from existing ones using `pipe` **function** (as opposed to `pipe` method). TypeScript 3.4 will certainly improve typing in such scenarios.

Other examples include Redux (`compose` for middleware) and Reselect.

## Summary

Introduction of _propagated generic type arguments_ has significany consequences for pointfree style programming. Before this update, using libraries such as `ramda` or `lodash/fp` with TypeScript was really cumbersome - you had to explicitly provide type arguments to certain calls which made the code far less readable. 

**TL;DR**: _Propagated generic type arguments_ pave the way for wider adoption of functional programming techniques in TypeScript.

---

Cover photo: <a href="https://pixabay.com/pl/users/JamesDeMers-3416/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=82592"> JamesDeMers</a> from <a href="https://pixabay.com/?utm_source=link-attribution&amp;utm_medium=referral&amp;utm_campaign=image&amp;utm_content=82592"> Pixabay</a>.
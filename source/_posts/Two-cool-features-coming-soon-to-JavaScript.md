---
title: Two cool features coming soon to JavaScript
date: 2019-09-08 13:29:52
tags:
  - javascript
  - es-next
  - tc39
image: /images/posts/chain.jpg
---

Recently two TC39 proposals have advanced to Stage 3.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">WE JUST MOVED OPTIONAL CHAINING IN JS TO STAGE 3 🎉🎉🎉🎉🎉🎉🎉</p>&mdash; Daniel Rosenwasser (@drosenwasser) <a href="https://twitter.com/drosenwasser/status/1154456633642119168?ref_src=twsrc%5Etfw">July 25, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Today I got to present nullish coalescing at TC39 and it progressed to stage 3! The cherry on top? <a href="https://twitter.com/rkirsling?ref_src=twsrc%5Etfw">@rkirsling</a> already has a patch out for it in JavaScriptCore! <a href="https://t.co/o3jHs2Ieo9">https://t.co/o3jHs2Ieo9</a></p>&mdash; Daniel Rosenwasser (@drosenwasser) <a href="https://twitter.com/drosenwasser/status/1153906097431777280?ref_src=twsrc%5Etfw">July 24, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

What it means to us, developers, is that two new exciting language features will soon become part of the ECMAScript standard.

Let's have a quick look at these additions and see how to take advantage of them.

## What's the deal with TC39 proposals?

[TC39](https://tc39.es) is a group of people that drives the development of the ECMAScript (the standard of which JavaScript language is an implementation). They meet regularly to discuss proposals of new language features. Every proposal goes through a number of stages. Once it reaches Stage 4, it is ready to be included in the next version of the ECMAScript standard.

When a proposal reaches Stage 3, it is already quite mature. The specification has been approved and is unlikely to change. There might already be some browsers implementing the new feature. While Stage 3 proposal is not guaranteed to become part of the standard, it's very likely to.

The two proposals we're looking at are:

- [Optional Chaining for JavaScript](https://github.com/tc39/proposal-optional-chaining)
- [Nullish Coalescing for JavaScript](https://github.com/tc39/proposal-nullish-coalescing)

## Optional chaining

Optional chaining aims to provide nice and short syntax for a very common pattern: accessing a nested property of an object in a safe way.

```javascript
const customers = [
  {
    name: "John",
    company: {
      name: "Acme",
      address: "London"
    }
  },
  {
    name: "Jane",
    company: {
      address: "New York"
    }
  },
  {
    name: "Judith"
  }
];
```

This array contains objects representing customers. They all follow a similar structure, but some of the properties are optional. Let's say we'd like to iterate over the array and print the company name in upper case for each customer.

```javascript
for (let customer of customers) {
  console.log(customer.company.name.toUpperCase());
}
```

As you might have guessed, the above code is not safe. It will result in runtime errors for the second and the third array elements. We can fix it by using the following popular pattern.

```javascript
console.log(
  customer &&
    customer.company &&
    customer.company.name &&
    customer.company.name.toUpperCase()
);
```

Logical _and_ operator (`&&`) in JavaScript behaves differently from most programming languages. It works on any value type, not only booleans. `a && b` translates to: if `a` is _falsy_ (can be converted to `false`), return `a`. Otherwise, return `b`.

Unfortunately, this solution is rather verbose. There is a lot of repetition and it gets worse the deeper the objects are nested. What's more, it checks for a value to be _falsy_, not `null` or `undefined`. Therefore, it would return `0` for the following object, while it might be preferable to return `undefined` instead.

```javascript
{
  name: "John",
  company: {
    name: 0,
  }
}
```

Optional chaining comes to the rescue! With this new feature, we can shorten the above piece to a single line.

```javascript
customer?.company?.name?.toUpperCase();
```

The `customer?.company` expression will check whether `customer` is `null` or `undefined`. If this is the case, it will evaluate to `undefined`. Otherwise, it will return `company`. In other words, `customer?.company` is equivalent to `customer != null ? customer : undefined`. The new `?.` operator is particularly useful when chained, hence the name (optional _chaining_).

Be careful when replacing existing `&&` chains with `?.` operator! Bear in mind the subtle difference it treatment of falsy values.

## Nullish coalescing

The second proposal introduces `??` operator which you can use to provide a default value when accessing a property/variable that you expect can be `null` or `undefined`.

But hey, why not simply use `||` for this? Similarly to `&&`, logical _or_ can operator on non-boolean values as well. `a || b` returns `a` if it's truthy, or `b` otherwise.

However, it comes with the same problem as `&&` - it checks for a _truthy_ value. For example, an empty string (`''`) will not be treated as a valid value and the default value would be returned instead.

```javascript
const customer = {
  name: "John",
  company: {
    name: ""
  }
};
customer.company.name || "no company"; // === 'no company'
```

Nullish coalescing operator can be nicely combined with optional chaining.

```javascript
(customer?.company?.name ?? "no company").toUpperCase();
```

While the benefit of optional chaining is clear (less verbose code), nullish coalescing is a little bit more subtle. We've all been using `||` for providing a default value for a long time. However, this pattern can potentially be a source of nasty bugs, when a falsy value is skipped in favour of the default value. In most cases, the semantics of `??` is what you're actually looking for.

## How can I use it?

Since those proposals have not reached Stage 4 yet, you need transpile the code that uses them (for example with Babel). You can play with [Babel's on-line REPL](https://babeljs.io/repl) to see what do they get compiled to.

At the moment of writing, optional chaining is [available in Chrome behind a feature flag](https://www.chromestatus.com/feature/5748330720133120).

Optional chaining will also be available in the upcoming [TypeScript 3.7 release](https://github.com/microsoft/TypeScript/issues/16#issuecomment-515160784)!

## Summary

Recent ECMAScript versions didn't bring many syntactical additions to the language. It's likely to change with the next edition. Some people say that JavaScript is getting bloated. I personally think that these two pieces of syntactic sugar are long overdue, as they've been available in many modern programming languages and they address real-life, common development scenarios.

What do you think? 😉

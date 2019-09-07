---
title: Why you should enable `strictNullChecks` in TypeScript?
date: 2018-12-22 15:13:07
tags:
  - typescript
  - static typing
---

I've been familiar with the `strictNullChecks` flag in TypeScript for a long time. However, it wasn't until recently that I had a chance to work with a huge codebase with `strict` mode enabled. Although I was aware of the benefits of this flag, I didn't expect it to be as awesome as it turned out to be in reality.

Read on to learn why you should definitely enable this flag in the project you're working on.

## What does it do?

For those of you who know what `strictNullChecks` does, you can safely skip to the next paragraph.

The purpose of `strictNullChecks` is to help you write safer code. It achieves that by pointing out places where you could be forgetting that a variable is `null` or `undefined`.

Imagine this small example:

```typescript
const appDiv: HTMLElement = document.getElementById('app');
appDiv.innerHTML = `<h1>TypeScript Starter</h1>`;
```

It looks fine. However, what if there were no element with id `app`? In such case `document.getElementById` would return `null` and we would get a `TypeError: appDiv is null` error. We would learn about the mistake only at runtime. But the purpose of TypeScript is to help you find mistakes during compile time!

Let's enable `strictNullChecks`. Now your IDE will highlight `appDiv` with the following message: 

```
Type 'HTMLElement | null' is not assignable to type 'HTMLElement'.
```

What this means is that the return type of `document.getElementById` is `HTMLElement | null` and we're trying to assign it to a constant with type `HTMLElement`. TypeScript doesn't allow this because the target type is narrower then the source type.

Enabling `strictNullChecks` changed the type of `document.getElementById` to `HTMLElement | null` instead of simply `HTMLElement`. In other words, this **type is now more honest and closer to the truth**. It admits that a `null` value can also be returned from this method. By doing this, it forces you to handle such case.

If we change the type of `appDiv` to `HTMLElement | null`, the next line will compile with the following error:

```
Object is possibly 'null'.
```

We cannot access a property of `appDiv` because it could be a `null`. TypeScript forces us to consider the `null` case. How to address it? It depends. Maybe you need to create this `app` element. Or maybe in such case it's better to throw an exception? The simplest solution would be to not do anything in case of falsy `appDiv`.

```typescript
const appDiv: HTMLElement | null = document.getElementById('app');
if (appDiv) {
    appDiv.innerHTML = `<h1>TypeScript Starter</h1>`;
}
```

This works because of TypeScript's great feature called **type guards**. The compiler is able to figure out that the type of `appDiv` *inside* the `if` is `HTMLElement` instead of `HTMLElement | null` thanks to some static code analysis.

## How does it work?

What exactly changed that allowed the compiler to work this way?

By default (with `strictNullChecks` **disabled**), the compiler behaves as if every type you use in your code was implicitly replaced with its union with `null` and `undefined`.

In other words, `null` and `undefined` values are part of every type. Therefore, typing something as `HTMLElement | null | undefined` was redundant since `HTMLElement` already included `null` and `undefined`.

Converesly, with `strictNullChecks` **enabled**, 


### Objects received from the backend

### Uninitialized properties (`strictPropertyInitialization`)

### Exhaustiveness checks

### React example - optional property of object passed as prop
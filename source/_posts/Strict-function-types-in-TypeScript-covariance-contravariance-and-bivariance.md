---
title: 'Strict function types in TypeScript: covariance, contravariance and bivariance'
date: 2019-05-30 21:45:55
tags:
   - typescript
   - 'strict mode'
image: /images/posts/strictFunctionTypes.jpg
---

Let's talk about one of the less well known strict type checking options - `strictFunctionTypes`. It helps you avoid another class of bugs, and it's also an excellent opportunity to learn about some fundamental computer science concepts: **covariance**, **contravariance**, and **bivariance**.

Strict function type checking was introduced in TypeScript 2.6. Its definition in TypeScript documentation refers to an enigmatic term: **bivariance**.

> Disable bivariant parameter checking for function types.

## What bugs can `strictFunctionTypes` catch?

First of all, let's see an example of a bug that can be caught by enabling this flag.

In the following example, `fetchArticle` is a function that accepts a callback to be executed after an article is fetched from some backend service. 

```typescript
interface Article {
    title: string;
}

function fetchArticle(onSuccess: (article: Article) => void) {
    // ...
}
```

Interestingly, TypeScript with default settings compiles the following code without errors.

```typescript
interface ArticleWithContent extends Article {
    content: string;
}

fetchArticle((r: ArticleWithContent) => {
    console.log(r.content.toLowerCase());
});
```

Unfortunately, this code can result in a runtime error. The function passed as a callback to `fetchArticle` only knows how to deal with with a specific subset of `Article` objects - those that also have `content` property.

However, `fetchArticle` can fetch all kinds of articles - including those that only have `title` defined. In such case, `r.content` is undefined, and runtime exception is thrown.

```
TypeError: undefined is not an object (evaluating 'r.content.toLowerCase')
```

Fortunately, enabling `strictFunctionTypes` results in a **compile-time error**. A compile-time error is always better than a runtime error, as it surfaces before users run your code.

```
Argument of type '(r: ArticleWithContent) => void' is not assignable to parameter of type '(article: Article) => void'.
```

## Covariance, contravariance, and bivariance

If you just wanted to learn what `strictFunctionTypes` does, you can stop reading right now. However, I encourage you to follow along and learn some background behind this check.

First, let's introduce a type to represent generic single-argument callbacks.

```typescript
type Callback<T> = (value: T) => void;

function fetchArticle(onSuccess: Callback<Article>) {
    // ...
}

declare const callback: Callback<ArticleWithContent>;
fetchArticle(callback); 
```

The reason `fetchArticle` shouldn't accept `callback` is that the callback is too specific. It only works on a subset of things that can be fed into it.

The type of `callback` should not be assignable to the type of `onSuccess` parameter. 

In other words, the fact that `ArticleWithContent` is assignable to `Article` does *not* imply that
`Callback<ArticleWithContent>` is assignable to `Callback<Article>`. If such implication were true, `Callback` type would be **covariant**. 

In our case, the opposite is true - `Callback<Article>` is assignable to `Callback<ArticleWithContent>`. That's because a callback that can handle all articles is also able to handle `ArticleWithContent`. Therefore, `Callback` is **contravariant**.

If both implications were true at the same time, then `Callback` would be **bivariant**.

Let's now revisit the definition of `strictFunctionTypes`.

> Disable bivariant parameter checking for function types.

Does it make sense now? With the check enabled, function type parameter positions are checked contravariantly instead of bivariantly.

On a side note, some function types are excluded from strict function type checks - e.g., function arguments to methods and constructors are still checked bivariantly.

## Summary

Wrapping up, `strictFunctionTypes` is a useful compiler flag that helps you catch a class of bugs related to passing function arguments, such as callbacks. 

The concept behind this flag is contravariance, which is a property of a type (type constructor, strictly speaking) that describes its assignability with respect to its type argument.

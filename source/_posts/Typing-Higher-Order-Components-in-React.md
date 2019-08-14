---
title: Typing Higher Order Components in React
date: 2019-08-14 23:12:18
tags:
    - typescript
    - generics
    - react
    - functional programming
image: /images/posts/archive.jpg
---

Some time ago [I wrote about](https://codewithstyle.info/TypeScript-3-4-hidden-gem-propagated-generic-type-arguments/) generic type arguments propagation feature added in TypeScript version 3.4. I explained how this improvement makes point-free style programming possible in TypeScript. 

As it turns out, there are more cases in which propagation of generic type arguments is desirable. One of them is passing a generic component to a Higher Order Component in React.

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">.<a href="https://twitter.com/orta?ref_src=twsrc%5Etfw">@orta</a> Do you know how to Propagate Generics across higher-order-components?<br><br>I have a generic component A, whose props is type Props = PubProps&lt;B&gt; &amp; InjectedProps<br>I have a `bind()`-HoC that injects InjectedProps<br><br>I want return value from bind:<br>APrime&lt;B&gt; accepting PubProps&lt;B&gt;</p>&mdash; Frederic Barthelemy (@fbartho) <a href="https://twitter.com/fbartho/status/1154542230641623040?ref_src=twsrc%5Etfw">July 26, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

_The post is inspired by the problem [Frederic Barthelemy](https://twitter.com/fbartho) tweeted about and asked me to have a look at._

## Higher Order Components

I'm not going to give a detailed explanation, as there are already plenty to be found on the internet. **Higher Order Component (HOC)** is a concept of the React framework that lets you abstract cross-cutting functionality and provide it to multiple components.

Technically, HOC is a function that takes a component and returns another component. It usually augments the source component with some behavior or provides some properties required by the source component.

Here is an example of a HOC in TypeScript:

```typescript
const withLoadingIndicator = 
    <P extends {}>(Component: ComponentType<P>): ComponentType<P & { isLoading: boolean }> => 
        ({ isLoading, ...props }) =>
            isLoading 
                ? <span>Loading...</span> 
                : <Component {...props as P} />;
```

As you can deduce from the type signature, `withLoadingIndicator` is a function that accepts a component with `P`-shaped properties and returns a component that additionally has `isLoading` property. It adds the behavior of displaying loading indicator based on `isLoading` property.

![HOC type](/images/posts/hoc.png)

## Problem: passing a generic component to a HOC

So far so good. However, let's imagine that we have a **generic component** `Header`:

```typescript
class Header<TContent> extends React.Component<HeaderProps<TContent>> { }
```

...where `HeaderProps` is a generic type that represents `Header`'s props given the type of associated content (`TContent`):

```typescript
type HeaderProps<TContent> = {
    content: TContent;
    title: string;
}
```

Next, let's use `withLoadingIndicator` with this `Header` component.

```typescript
const HeaderWithLoader = withLoadingIndicator(Header);
```

The question is, what is the inferred type of `HeaderWithLoader`? Unfortunately, it's `React.ComponentType<HeaderProps<unknown> & { isLoading: boolean; }>` in TypeScript 3.4 and later or `React.ComponentType<HeaderProps<{}> & { isLoading: boolean; }>` in previous versions. 

As you can see, `HeaderWithLoader` is **not** a generic component. In other words, generic type argument of `Header` was **not propagated**. Wait... doesn't TypeScript 3.4 introduce generic type argument propagation?

## Solution: use function components!

Actually, it does. However, it only works for **functions**. `Header` is a generic class, not a generic function. Therefore, the improvement introduced in TypeScript 3.4 doesn't apply here ☹️

Fortunately, we have **function components** in React. We can make type argument propagation work if we limit `withLoadingIndicator` to only work with function components.

Unfortunately, we cannot use `FunctionComponent` type since it is defined as an interface, not a function type. However, a function component is nothing else but a generic function that takes props and returns `React.ReactElement`. Let's define our own type representing function components.

```typescript
type SimpleFunctionComponent<P> = (props: P) => React.ReactElement;

declare const withLoadingIndicator: 
    <P>(Component: SimpleFunctionComponent<P>) => 
        (SimpleFunctionComponent<P & { isLoading: boolean }>);
```

_By using `SimpleFunctionComponent` instead of `FunctionComponent` we loose access to properties such as `defaultProps`, `propTypes`, etc., which we don't need anyway._

Obviously, we need to change `Header` to be a function component, not a class component:

```typescript
declare const Header: <TContent>(props: HeaderProps<TContent>) => React.ReactElement;
```

_We wouldn't be able to use `FunctionComponent` here anyway, since `Header` is a generic component_.

Let's now take a look at the inferred type of `HeaderWithLoader`. It's...

```
<TContent>(props: HeaderProps<TContent> & { isLoading: boolean }) => React.ReactElement
```

...which looks very much like a generic function component!

Indeed, we can use `Header` as a regular component in JSX:

```typescript
class Foo extends React.Component {
    render() {
        return (
            <HeaderWithLoader 
                title="Hello" 
                content={12345} 
                isLoading={false} />
        );
    }
}
```

Most importantly, `HeaderWithLoader` is typed correctly!

## Summary

As you can see, typing HOCs in React can get tricky. The proposed solution is really a workaround - ideally, TypeScript should be able to propagate generic type arguments for all generic types (not only functions). 

Anyway, this example demonstrates how important it is to stay on top of the features introduced in new TypeScript releases. Before version 3.4, it wouldn't be even possible to get this HOC typed correctly.
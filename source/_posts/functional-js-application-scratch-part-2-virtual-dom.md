---
title: Functional JS application from scratch - part 2 - virtual DOM
url: 821.html
id: 821
categories:
  - JavaScript
  - Web
date: 2018-09-30 19:17:11
tags:
  - javascript
  - functional programming
  - vanilla js
icon: fas fa-hammer
---

My [previous post](https://codewithstyle.info/functional-javascript-app-scratch/) described how to create a very basic web application following the principles of functional programming. That's fine, but I bet you're not building _basic_ apps. **How to scale this approach?** This (and the following) post will present some techniques you could use to solve common problems encountered when creating more complex applications.

Let me remind you that the end goal is not to convince you to ditch all JavaScript frameworks and build your own instead. My point is to **explain the reasoning behind commonly used patterns and how they relate to functional programming**. Source code for this article is available [here](https://github.com/miloszpp/functional-climbs/tree/part-2-virtual-dom).

Limiting DOM updates
--------------------

Our little _framework_ relies on `view` function which translates state into DOM tree. The function is invoked on every state update. This means that **on every state change we need to re-create the whole DOM tree** and have it re-rendered by the browser. In a complex application with multiple actions and huge DOM tree this could have a huge impact on performance. Below you can find a screenshot illustrating the problem. The whole `div` is updated even though clicking _Complete_ should only affect two table rows. 

![](/images/2018/09/fp-app-without-vdom-small-1024x423.gif) 

Basically, we'd like to limit the amount of unnecessary DOM-related work. On the other hand, we want our code to stay declarative and functional so we have to avoid direct, imperative DOM manipulation.

Introducing **virtual DOM**
---------------------------

**Virtual DOM** is the answer to our problems! It is a clever technique where instead of creating actual DOM objects you operate on _virtual_ DOM elements. Operations on virtual nodes are much faster than on actual nodes since there is no browser API involved. Obviously, at some point we need to update the actual DOM tree. Here is how we will do this:

*   `view` function will return a virtual DOM tree
*   on every state update (every `app` invocation) we will compare the result of the `view` call with the previous tree
*   the comparison results in a set of `patches` that represent minimal changes to the DOM
*   `patches` can be applied to the actual DOM tree; only relevant parts of the DOM are updated, not the whole tree

Below you can see the difference after enhancing the application with virtual DOM. Note that only relevant parts of the DOM are highlighted. 

![](/images/2018/09/fp-app-with-vdom-small-1024x449.gif)

Show me the code
----------------

Let's rework our application to take advantage of virtual DOM.

### View function

The first step is to adjust the `view` function to return virtual nodes instead of real DOM nodes. We are not going to implement the virtual DOM mechanism itself. It's a highly non-trivial task and not in the scope of this article. Instead, let's use one of existing virtual DOM libraries. Our library of choice is simply called [virtual-dom](https://github.com/Matt-Esch/virtual-dom). The best thing about it is that it's compatible with `hyperscript-helpers`. Remember how we wrapped `hyperscript` with `hyperscript-helpers` so that we were able to use functions such as `div`, `h2`, `table`, etc.? This extra level of indirection will prove enormously useful now. Our `view` function will continue using these functions. However, they will proxy to `virtual-dom` instead of `hyperscript` resulting in virtual DOM nodes instead of real DOM nodes.


```typescript
    import h from 'virtual-dom/h';
    import hh from 'hyperscript-helpers';
    
    const { table, tr, td, th, div, h2, button } = hh(h);
```

That's it! There are no more changes to `view` function!

### Engine

The next (and last) step is to adjust the `app` function. It's going to get a bit more complex. Before, all we had to do was to replace the _old_ DOM tree with the _new_ tree. However, `view` returns a virtual tree now. We need to compare the _new_ tree and with _old_ one using `diff` function provided by the library. The comparison will return a set of `patches` which can be later applied on the actual DOM tree. The above procedure describes what happens on state update. However, we need an initial DOM tree to begin with! We can get one from the initial virtual tree by calling `createElement`, also provided by the library. Below you can find the update code.

```typescript
    import diff from 'virtual-dom/diff';
    import patch from 'virtual-dom/patch';
    import createElement from 'virtual-dom/create-element';
    
    function app(state, previousView = null) {
        const updatedView = view(dispatch, state);
    
        if (previousView === null) {
          const updatedViewDom = createElement(updatedView);
          rootNode.appendChild(updatedViewDom);
        } else {
          const patches = diff(previousView, updatedView);
          patch(rootNode.children[0], patches);
        }
    
        function dispatch(action) {
            const nextState = reducer(state, action);
            app(nextState, updatedView);
        }
    }
    
    app(initialState);
```

The `app` function accepts a new parameter called `previousView`. We need it to be able to compare updated virtual DOM with the previous version. When `previousView` is `null`, it means that `app` is called for the first time (with `initialState`) and that there is no real DOM tree in place yet. Therefore, we call `createElement` and append the result to `rootNode`. When `previousView` is not empty, we should compare it with the new virtual tree (`updatedView`) and apply patches on `rootNode`'s first child (because we initially attached the whole tree to `rootNode`). Obviously, this part is not functional code. Patching the DOM is an imperative operation with side effects. However, this part of the code wasn't pure in the first place. What's important is that **we've managed to preserve the purity of the rest of the code**.

Virtual DOM in real world
-------------------------

The concept of virtual DOM is instrumental in React framework. This is actually what made React famous for its performance. By the way, if you are familiar with React, you might have noticed similarities between our application and the framework. Indeed, **our `view` function is nothing less than a React functional component**! It's interesting to see how other frameworks deal with limiting the amount of DOM operations while maintaining declarativeness. For example, Angular takes a different approach based on **change detection**. You can read more about it in [one of my posts](https://codewithstyle.info/change-detection-angular-versus-angularjs/). This comparison between those two mechanisms seems very interesting to me and I asked a [question on Reddit specifically about it](https://www.reddit.com/r/Angular2/comments/8ytfc1/reacts_virtual_dom_vs_angulars_change_detection/). I've got an amazing reply from Rob Wolmard which details the pros and cons of both approaches:

> The tradeoff (and part of the philosophy Angular is built around) is that templating allows Angular to deeply understand a template, and generate highly optimized code, in both pure CPU cycles, but low memory consumption and garbage collection. The flexibility of being able to return whatever from a JSX-style `render()` function means the framework has to be able to handle whatever, and each time a new virtual DOM representation is created, it can consume a fair amount of memory - especially important on low end devices.

Summary
-------

In this post you've learned what virtual DOM is and how frameworks can use it to optimize performance while remaining declarative. I believe that this example nicely illustrates how trying to build your own framework can push you to learn new things and understand how existing frameworks work underneath.
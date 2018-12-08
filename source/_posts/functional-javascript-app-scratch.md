---
title: Functional JavaScript app from scratch
url: 790.html
id: 790
categories:
  - JavaScript
  - Web
date: 2018-08-15 10:11:37
tags:
  - functional programming
  - vanilla js
icon: fas fa-hammer
---

I've been writing a lot about specific functional concepts and their implementation in JavaScript and TypeScript. In this article, I'd like to show you how to put some of these ideas together and **create a working application based on functional programming principles**. The application we're about to build is called _Functional Climbs_. It presents a list of climbing routes and allows the user to mark some of them as completed. 

I'm not going to use any framework but there will be some helper libraries involved. The source code is available [here](https://github.com/miloszpp/functional-climbs).

State
-----

Every application has a **state**. It can be distributed across fields in multiple objects (components, controllers, etc.) or it can be centralized. In the centralized approach, the state is an object that stores the information the application needs to function. In our case, the state will contain an array of objects, each one representing a climb.

```typescript
    export const initialState = {
        climbs: [
            {
                id: 1,
                name: 'Mount Blanc',
                elevation: 4808,
                difficulty: 'medium',
                completed: false
            },
            {
                id: 2,
                name: 'Matterhorn',
                elevation: 4478,
                difficulty: 'hard',
                completed: false
            }
        ]
    };
```

Feel free to extend the list and add some more climbs!

From State to View
------------------

Now that we know the structure of the state, it's time to create a function that produces a DOM tree based on the state. Since we're doing functional programming, this function should, of course, be **pure**. How to create the DOM tree in a functional way? The standard DOM API is not very functional - it relies on global objects and mutations. Instead, we're going to use two libraries - [hyperscript](https://github.com/hyperhype/hyperscript) and [hyperscript-helpers](https://github.com/ohanhi/hyperscript-helpers). _Hyperscript_ makes it possible to create DOM trees in a declarative way. Instead of manipulating nodes you simply declare what you'd like the document to look like:

```typescript
    var h = require('hyperscript')
    h('div#page',
      h('div#header',
        h('h1.classy', 'h', { style: {'background-color': '#22f'} })),
      h('div#menu', { style: {'background-color': '#2f2'} },
    ...
```

With _hyperscript-helpers_ the task becomes even easier as instead of using `h` function all the time you can use functions named after HTML tags

```typescript
    h('div') ---> div()
    h('section#main', mainContents) ---> section('#main', mainContents)
```

If you don't feel like using JS functions instead of HTML, you can achieve the same results with JSX.

Implementing View function
--------------------------

The role of the **view function** is simple - it has to translate state into a DOM tree by calling various `hyperscript-helpers` functions. Most of these functions accept three parameters:

*   CSS selector that can be used to set `class` and `id` of the element
*   an object with properties (attributes)
*   an object or an array containing children

It's a good idea to break down the `view` function into smaller functions representing different **components** of the UI. You might put the implementation of this function inside a separate `view.js` file. Let's take the top-down approach and start with the root function. We separate climbs into two arrays representing completed and remaining climbs. Next, we use the `climblist` component to display them. CSS classes are referring to Bootstrap.

```typescript
    import h from 'hyperscript';
    import hh from 'hyperscript-helpers';
    
    const { table, tr, td, th, div, h2, button } = hh(h);
    
    export function view(state) {
        const completedClimbs = state.climbs.filter(c => c.completed);
        const remainingClimbs = state.climbs.filter(c => !c.completed);
    
        return div('.container', null, [ 
            h2('.mt-3', null, 'Remaining climbs'),
            climblist(remainingClimbs),
            h2('.mt-3', null, 'Completed climbs'),
            climblist(completedClimbs),
        ]);
    }
```

Component `climblist` is simply an HTML table. We `map` every climb inside the state to `climblistRow` component. As an exercise, you can add a static table header to this function!

```typescript
    function climblist(climbs) {
        return table(
            '.climblist.table',
            [
                ...climbs.map(climblistRow)
            ]
        );
    }
```

Finally, `climblistRow` consists of cells containing data from `climb` objects. The last column contains a button that will toggle "completeness" of a climb. For now, it will not do anything.

```typescript
    function climblistRow(climb) {
        const toggleLabel = climb.completed ? 'Uncomplete' : 'Complete';
        return tr(
            '.climblist__row',
            null,
            [ 
                td('.climblist__cell', null, climb.name),
                td('.climblist__cell', null, climb.elevation),
                td('.climblist__cell', null, climb.difficulty),
                td(
                    '.climblist__cell', 
                    null, 
                    button(
                        '.btn.btn-primary', 
                        null, 
                        toggleLabel
                    )
                )
            ]
        );
    }
```

You are now ready to put the view function into action. Create an `index.html` file with the following body:

```html
    <body>
        <div id='root'>Loading...</div>
    </body>
 ```

Now all you need to do is to run `view` function on `initialState` and attach the result to the `root` node. Create and `index.js` file with the following content:

```typescript
    import { view } from './views';
    
    const rootNode = document.getElementById('root');
    const currentNode = rootNode.childNodes[0];
    const newNode = view(initialState);
    
    rootNode.replaceChild(newNode, currentNode);
```
    

To make this all work together you will need a module bundler. Check out the webpack config in the [source code](https://github.com/miloszpp/functional-climbs). When you launch the application you should see a table displaying climbs defined in the initial state. 

![](/images/2018/08/Screen-Shot-2018-08-13-at-20.50.47-1024x321.png)

Let's have some _action_
------------------------

So far our application is rather boring. Toggle buttons don't do anything interesting. It's time to change it! As you know, the most important part of the application is the state object which is _the golden source of truth_. So if we want something to happen then we need to modify the state. But hey, it's functional programming - we **can not mutate the state**! Instead, we will return a fresh, updated copy of the state and re-generate the DOM based on it. Let's create a function called `reducer` which takes a state object and an action object and returns a new state object. But what's the **action** object? It's simply a piece of data representing some event in the application. We will only have a single action that will be triggered when the user clicks on the toggle button. It has to store the id of the climb that is being toggled. Let's create `actions.js` file with the following contents:

```typescript
    export const TOGGLE_COMPLETED_ACTION = 'TOGGLE_COMPLETED_ACTION';
    
    export function toggleCompleted(climbId) {
        return {
            type: TOGGLE_COMPLETED_ACTION,
            payload: climbId
        };
    }
```

It's a common convention to have these two fields inside action objects. `type` determines what action is it and `payload` contains details of the event. In our case `payload` is simply `climbId` but it could be any object. Next, we will write the `reducer` function. Given the state and the `TOGGLE_COMPLETED_ACTION` action, it needs to return a new state object where the climb with provided `id` is replaced with a new climb object with `completed` field flipped.

```typescript
    export function reducer(state, action) {
        switch (action.type) {
            case TOGGLE_COMPLETED_ACTION:
                return { 
                    ...state, 
                    climbs: state.climbs.map(c => c.id === action.payload ? { ...c, completed: !c.completed } : c)
                };
            default:
                return state;
        }
    }
```

Note how we make sure not to mutate the `state` object. _Spread operator_ and `map` method are pretty useful in this scenario. Next, we need a way to trigger this new action. Let's assume for now that `view` function accepts another parameter called `dispatch`. It will be a function and it will be used to trigger actions. We need to pass this parameter all the way down to `climblistRow` component where it can be hooked to `onclick` property of the toggle button.

```typescript
    button(
        '.btn.btn-primary', 
        { onclick: () => dispatch(toggleCompleted(climb.id)) }, 
        toggleLabel
    )
```

Make sure to put `dispatch` as the first parameter of all view functions. It will allow you to do **partial application** using `curry` function (which you can get from [Ramda](https://ramdajs.com/)).

```typescript
    ...climbs.map(curry(climblistRow)(dispatch))
```

Putting it all together
-----------------------

It's time to create some machinery so that action results in DOM updates. This is the only impure fragment of the codebase! Replace the contents of `index.js` with the following code.

```typescript
    const rootNode = document.getElementById('root');
    
    function app(state) {
        const updatedView = view(dispatch, state);
        const currentView = rootNode.childNodes[0];
    
        rootNode.replaceChild(updatedView, currentView);
    
        function dispatch(action) {
            const nextState = reducer(state, action);
            app(nextState);
        }
    }
    
    app(initialState);
```

The `app` function _applies_ provided state object to the DOM. What's more, it defines the `dispatch` function which is used to trigger actions. This function uses `reducer` to apply the action on existing state and recursively passes the new state object to `app`. Finally, we call `app` with `initialState`. And that's it! Run the application and try out toggle buttons - they should work now. 

![](/images/2018/08/functional-climbs1-1.png)

Summary
-------

You've just created a working application which is as functional as it possibly can. Of course it has some impure parts - every application should have some side effects, otherwise, it wouldn't be very useful! The trick is to minimize and isolate the impure parts. For those of you familiar with Redux or MobX this approach can feel very familiar. Indeed, we've implemented a simplistic version of a state management framework. I totally suggest using a real framework instead but for educational purposes, it's a good idea to see how it works underneath. You might have concerns about performance. Re-creating the whole DOM on every tiny state change doesn't sound optimal. The answer to this is **virtual DOM**. There are libraries that allow you to create virtual DOM nodes that can later be compared and only small, incremental changes to the DOM are executed. This is exactly how React works! `hypescript-helpers` are compatible with some virtual DOM libraries. I hope the article was clear enough and that now it's clearer how to apply functional programming principles in real life!
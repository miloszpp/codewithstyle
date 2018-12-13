---
title: Better RxJS code with pointfree style
url: 874.html
id: 874
categories:
tags:
  - javascript
  - functional programming
  - pointfree style
image: /images/2018/11/business-3421076_1920.jpg
icon: fas fa-dot-circle
date: 2018-12-13 23:00:00
---

What is pointfree style?
------------------------

It's easiest to explain pointfree style by showing a small example. Let's say we have this array of objects representing mountains:

```javascript
    const mountains = [
      { name: 'Mount Everest', height: 8848 },
      { name: 'Mont Blanc', height: 4808 },
      { name: 'Kazbek', height: 5033 },
    ];
```

As you know, you can use array methods such as `map` or `reduce` to run some calculations on this array.

```javascript
    const names = mountains.map(mountain => mountain.name);
    const totalHeight = mountains
        .map(mountain => mountain.height)
        .reduce((result, height) => result + height, 0);
```

Both of these methods are *higher-order functions* - they accept a function as an parameter. In both cases we create an anonymous function using arrow function syntax and pass it as an argument. Now let's make those two lines more pointfree.

```javascript
    mountains.map(prop('name'));
    mountains.map(prop('height')).reduce(add, 0);
```

What happened here? I replaced arrow functions with small, self-explaining functions such as `prop` and `add`. These functions come from a library called [ramda](https://ramdajs.com/), but you could easily write them yourself. As you can imagine, `prop('name')` returns a function that reads `name` property from given object and `add` simply adds two numbers.

```javascript
    prop('name')({ name: 'Kazbek'}); // 'Kazbek'
    add(1, 3)); // 4
```

Pointfree style can be summarized as the following rule: _never mention the data_. In our example, _data_ refers to parameters of anonymous functions. Instead of using anonymous functions, we use small, very generic functions. We build applications by _composing_ these functions. This forces us to focus on **data tranformations** instead of on the data itself. 

BTW, _point_ in _pointfree_ doesn't refer to `.` that is part of method call syntax. It comes from category theory, where _points_ are elements of sets. _Pointfree_ in this context means that you focus on transformations (_morphisms_) between set elements (_points_) instead of on the _points_ themselves.

Partial application
-------------------

There are two very important concepts that make pointfree style possible. The first one is called _partial application_. I've already used partial application in this article. It happened when I called `prop('name')`. Logically, `prop` takes two arguments: a property name and an object:

```javascript
    prop('name', { name: 'Kazbek'});
```

However, we used it as if it was a single argument function: `prop('name')`. By doing this, we've partially applied `prop`. The result is a function that will _wait_ for the second argument and only evaluate once it receives it. 

But wait, if I only provided one argument to two argument function, it will still evaluate - `undefined` will be passed as the second argument, right? Well, it's true unless the function is _curried_. Currying is the process of taking a multi-argument function and transforming it into a single argument function. 

![](/images/2018/12/pointfree-Page-2.png)

Here is a simplified, multi-argument version of `prop` \- it can't be partially applied:

```javascript
    const prop = (propName, object) => object[propName];
```

And here is a curried version:

```javascript
    const prop = (propName) => (object) => object[propName];
```

Do you see the difference? The second version is indeed a single argument function which returns another single argument function. You don't need to curry your functions manually - you can use `curry` function from `ramda` instead.

```javascript
    const prop = curry((propName, object) => object[propName]);
```

Function composition
--------------------

Another concept essential for pointfree style is **function composition**. I wrote a [separate article](https://codewithstyle.info/deep-dive-pipe-function-rxjs/) about, so please take a look if you are not familiar with the concept.

Application in RxJS
-------------------

Ok, I promised you some RxJS code 😉 Let's start with a simple example. Upon clicking the `breedsFetchEl` button we want to fetch some data from [Dog API](https://dog.ceo/api) and present it in `breedsListEl` div. Easy peasy.

```javascript
    const click$ = fromEvent(breedsFetchEl, 'click');
    const breeds$ = click$.pipe(
      switchMap(() => 
        ajax.getJSON('https://dog.ceo/api/breeds/list/all')),
      map(result => Object.keys(result.message).join(', '))
    );
    breeds$.subscribe(breeds => breedsListEl.innerText = breeds);
```

Let's convert it to pointfree style. We will focus on the anonymous function inside `map`. Let's try to figure our what's going on there:

*   First, `message` property is read from `result`.
*   Next, its value is passed to `Object.keys` function.
*   Finally, we call `join` on the value returned by that call.

We don't like method calls in functional programming - they are not composable. Let's first replace the third step with a function call. We will use `join` from `ramda`.

```javascript
    map(result => join(', ', Object.keys(result.message)))
```

Property access also comes from the OOP word, so let's change it to a function call. This time, let's use `prop` from `ramda`.

```javascript
    map(result => join(', ', Object.keys(prop('message', result))))
```

Now this looks familiar. We have three nested function calls. We call the first one and pass the result to the second one. The result of the second one is passed as an argument of the third one. This is exactly what function composition is. Let's use `pipe` to replace the anonymous function with a function composed from these three functions.

```javascript
    map(
      pipe(
        prop('message'),
        Object.keys,
        join(', ')
      )
    )
```

![](/images/2018/12/pointfree.png)

But wait, `prop` and `join` are two argument functions and now we partially apply them! How is this possible? The answer to this is that `ramda` functions are really clever. They are curried by default. However, they do also work as expected when called with multiple arguments.

More advanced example
---------------------

Let's now take a look at a slightly more complex scenario. We have a text field (`dogNameInputEl`) and we want to show dog images (inside `dogImageEl`) based on breed names entered into this field. We want the image to update as you type, but we don't want to overload the server with request. Hence, we will only send the request if breed name's length exceeds 2 and we will debounce calls.

```javascript
    const nameInput$ = fromEvent(dogNameInputEl, 'input');
    
    const image$ = nameInput$.pipe(
      map(() => dogNameInputEl.value),
      filter(name => name.length >= 3),
      debounceTime(500),
      mergeMap(name => 
        ajax.getJSON(`https://dog.ceo/api/breed/${name}/images/random`)),
      map(result => result.message)
    );
    
    image$.subscribe(
      imageUrl => dogImageEl.setAttribute('src', imageUrl));
```

Cool, let's deal with this example line by line. The first operator maps `input` events to actual values typed into the text field.

```javascript
    map(() => dogNameInputEl.value)
```

We need a function that ignores its input and _always_ returns `dogNameInputEl`. Then we could pass its result to `prop('value')`. It turns out that there is such function in `ramda`. It's called... `always`!

```javascript
    map(pipe(always(dogNameInputEl), prop('value'))
```

Next line filters input values so that we only process those that are longer then 2. It boils down to function composition again. First, we calculate string length. You guessed right, there is a function called `length` in `ramda` that we can use. Second, we compare the result with 3. `ramda` has a function called `gte`. However, if we used it like this `gte(3)` the we would get a function that returns `true` if 3 is greater then or equal to its argument. Therefore, we need to swap `gte`'s arguments. `flip` is a function which does exactly that!

```javascript
    filter(pipe(length, (flip(gte))(3)))
```

There is no anonymous function in `debounce`, so we can skip it. Next, `mergeMap` transforms breed name to a response object retrieved from the server. Again, we can use function composition. First, we transform the name into an URL. Next, we pass the URL to `ajax.getJSON`. Transforming the name into an URL is achieved using template strings. We could compose this using `ramda`'s functions but the result would not be readable at all. In such case, it's better to create a helper function and simply use it inside `pipe`. It's not super pointfree, but code readability is more important (another approach would be to use order based string formatting library, such as `sprintf-js`).

```javascript
    const getImageUrl = 
      (name) => `https://dog.ceo/api/breed/${name}/images/random`;
    
    mergeMap(pipe(getImageUrl, ajax.getJSON))
```

Finally, the last line simple extracts `message` property from the result. I'm pretty sure you already know how to do that 😉 The end result will look like this:

```javascript
    const image$ = nameInput$.pipe(
      map(pipe(always(dogNameInputEl), prop('value'))),
      filter(pipe(length, (flip(gte))(3))),
      debounceTime(500),
      mergeMap(pipe(getImageUrl, ajax.getJSON)),
      prop('message')
    );
```

Summary
-------

What do you think about pointfree style? In my opinion, it's a great programming practice, but one that has to be applied with caution. In general, it makes your code more concise and eliminates the boilerplate of anonymous functions. More importantly, it forces you to think about your program as a composition of small, generic building blocks (functions). Pointfree style lets you experience the full power of functional programming!

However, pointfree shouldn't be applied at all cost. In some cases, it might make the overall readability of code much worse. What's more, if you're working in a team, you have to keep in mind that such style of writing code might be unfamiliar to others.

Don't hesitate to share your opinion about pointfree style in comments section!
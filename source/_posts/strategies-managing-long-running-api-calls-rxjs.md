---
title: Strategies for Managing Long-running API Calls with RxJS
url: 862.html
id: 862
categories:
  - JavaScript
  - TypeScript
  - Web
date: 2018-12-02 11:20:05
tags: ['rxjs', 'reactive programming']
icon: fas fa-clock
---

**This post has been originally published on [SumoLogic company blog](https://www.sumologic.com/blog/devops/long-running-api-queries/)**. 

It’s common for a modern single-page application (SPA) to fetch data from the server via a REST API call. The vast majority of web applications do this. There are, though, many challenges related to this approach, one of which is handling long-running queries. In order to ensure a great user experience, we can’t have the user wait four or five minutes to see the results of an action. 

This is often the case here at Sumo Logic, where, for instance, the user interface (UI) sends complex search queries to the backend. Depending on the query, processing might take a few minutes. In this article we will discuss different approaches for dealing with this issue. We’ll rely on the RxJS library to help us with this task because it’s perfect for dealing with complex, asynchronous flows.

Strategies for Dealing with Long-running Calls
----------------------------------------------

There are multiple approaches that can be taken and in this article I’ll discuss three of them. The list is here mostly for inspiration, as the solution for your specific problem will very likely depend on your use case and the design of your API. Here’s a quick summary of the different approaches I will discuss throughout this post:

*   Displaying a loading indicator – indicates to the user that a long-running query is currently being processed.
*   Showing partial results – splits the large query into smaller queries and combines the results on the fly.
*   Showing the best result within given time period – makes several parallel calls aiming for varying accuracy of the result and shows the best result we could achieve within a given timeframe.

Code examples below are simplified; in reality you also need to take care of error handling and unsubscribing.

Approach #1: Display a Loading Indicator
----------------------------------------

This is the most basic approach because we don’t really fix the problem, but rather simply improve user experience by indicating to the user that the query is being processed (or whatever long-running action is happening in your system). Let’s assume our task is to fetch a list of customers. Unfortunately, this API call is rather slow. In order to make sure that the user is aware of the fact that a query is being processed, we’ll show a loading spinner. Let’s say we already have the following function, which can fetch the list of customers from the backend. It returns an observable, which will emit once, when the server replies. If you’re using the fetch API, you can easily convert a promise to an observable using the from function.

```typescript
function fetchCustomers(): Observable<Customer[]> { ... }
```

Fetching the list is very likely initiated by a user action such as clicking a button. Let’s create a `click$` stream, which emits button clicks and then use `switchMap` to transform it into `customers$` stream, which will emit lists of customers retrieved from the server.

```typescript
    const click$ = fromEvent(buttonEl, 'click');
    const customers$ = click$.pipe(switchMap(() => fetchCustomers()), share());
```

As a next step, we’ll create a new stream that emits true whenever the loading spinner should be shown and false when it shouldn’t. We’ll do this by merging `click$` and `customer$` streams:

*   Event emitted by `click$` means that we should show the spinner so we’ll map it to true
*   Event emitted by `customer$` means that we should hide the spinner so we’ll map it to false

```typescript
    const isLoading$ = merge( 
      click$.pipe(mapTo(true)), 
      customers$.pipe(mapTo(false)), 
    );
```
    

Now, all that’s left is to subscribe to the stream and update the loading indicators visibility.

```typescript
    isLoading$.subscribe(isLoading => 
      loadingIndicatorEl.style.visibility = 
        isLoading ? 'visible' : 'hidden');
```
    

Approach #2: Show Partial Results
---------------------------------

The goal of this second approach is to improve the user experience by not making the user wait for the whole query to be processed, but rather to show something whenever some results are available. We’ll achieve this by splitting the long-running query into smaller queries. Of course this approach is based on some assumptions about our API:

*   It is possible to split a query into smaller queries.
*   Smaller queries will actually execute faster than large queries.

What do I mean by splitting the query into smaller ones? For example, instead of fetching the full list of customers at once, we might decide to fetch small portions of the list and combine them in the UI. Let’s see the code, and assume that we’re now working with the following function that can be parameterized by some offset. This offset can be used to decide which part of the list to fetch. Let’s also assume that the function will always fetch a fixed number of matching customers (e.g. 100).

```typescript
    function searchCustomersPaged(query: string, offset: number) 
      { ... }
```

We can start by creating an array of offsets and map it into queries. The first query will fetch customers from 0 to 99, the second will fetch customers from 100 to 199, and so on.

```typescript
    const offsets = [0, 100, 200, 300]; 
    const queries = offsets.map(offset => 
      searchCustomersPaged('some query', offset).pipe(startWith(null)));
```

Each stream will emit a null followed by the actual result. As a next step, we’ll combine those streams into a single stream which emits concatenated, non-empty results.

```typescript
    const result$ = combineLatestFun(queries).pipe( 
      map((results) => { 
        const nonNullResults = results.filter(r => r !== null); 
        return nonNullResults.reduce((acc, r) => [ ...acc, ...r ], []); 
      }) 
    );
```

We ended up with a single stream that will emit a growing list of customers, which we can show to the user in real time. This is a much nicer user experience then having to wait for the whole list to be fetched. Note: it’s important to keep in mind that browsers put limitations on the number of concurrent queries made to the same domain. It doesn’t make any sense to exceed this number.

Approach #3: Show the Best Result
---------------------------------

Finally, in this approach we’re going to fire parallel queries aiming for different accuracy of the result. We’ll then wait and, after some fixed amount of time, return the best (most accurate) result of those received so far. Quick shout out goes to one of my colleagues, **Omid Mortazavi**, who came up with the idea for this third approach. How does this translate to the customer search scenario? Let’s say that the API includes a parameter for specifying the level (precision) of search accuracy. A customer search with a lower accuracy will be faster but not as exhaustive as a search with a higher accuracy. We want to present the user with the best result yet we don’t want them to wait too long. We’ll therefore trigger several searches, of varying precision, and only wait a fixed amount of time. Similar to the previous approach, let’s start by creating an array of different accuracy levels and mapping them into queries.

```typescript
    const accuracyLevels = [5, 3, 1]; 
    const queries = accuracyLevels.map(level => 
      searchCustomers('some query', level).pipe(startWith(null)) 
    );
```
    

Next, let’s create a stream that will emit true after a fixed period of time elapses.

```typescript
    const timeoutElapsed$ = timer(10000).pipe(
      mapTo(true),
      startWith(false)
    );
```

Finally, we’ll combine all of the streams in queries with the `timeoutElapsed$` stream. The combined stream will emit whenever any of the source streams emit. The second parameter of `combineLatest` is a function in which we decide what to do when it happens. The logic is as follows:

*   Timeout not elapsed yet
    *   All queries finished so return the most accurate result
    *   Some queries not finished yet so return null as we still hope for a better result
*   Timeout elapsed
    *   Return the most accurate result

```typescript
    const result$ = timeoutElapsed$.pipe(
      combineLatest(queries, (isTimeoutElapsed, ...results) => {
        if (!isTimeoutElapsed) {
          const notReadyResult = results.find(result => result === null); 
          return notReadyResult ? notReadyResult : results[0]; 
        } else {
          return results.find(result => result !== null) || null;
        }
      }), 
      filter(result => result !== null), 
    );
```

Below you can find marble diagrams demonstrating this approach based on two concurrent queries. 

![](https://codewithstyle.info/wp-content/uploads/2018/11/Example-1_-1.png)

Example 1: one query finishes before timeout elapses

![](https://codewithstyle.info/wp-content/uploads/2018/11/Example_2.png)

Example 2: both queries finish before timer elapses

![](https://codewithstyle.info/wp-content/uploads/2018/11/Example_3.png)

Example 3: neither query finishes before timeout elapses

One final thought: if the API provides such an option, cancel any pending searches once we’ve presented the result to the user to avoid unnecessary backend work or network traffic. In scenarios demonstrated by the diagrams above there is no cancellation at all. Therefore, the `result$` stream emits multiple times, which might not be desirable.

Summary
-------

We’ve discussed three different approaches to improving user experience when dealing with long running API calls. While this list is by no means exhaustive and these techniques might need some adjustments based on your specific situation, I hope you’ve seen some of the power of functional-reactive programming with RxJS and can see other areas of your applications which can benefit from the possibilities it enables.
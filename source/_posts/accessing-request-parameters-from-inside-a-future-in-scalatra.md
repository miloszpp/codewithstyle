---
title: Accessing request parameters from inside a Future in Scalatra
tags:
  - scala
  - scalatra
url: 7.html
id: 7
categories:
  - Other topics
  - Quick solutions
  - Scala
date: 2016-01-21 21:47:00
---

[Scalatra](http://www.scalatra.org/) is an awesome, lightweight web framework for Scala. It's perfect for building REST APIs. One of its less known features is support for asynchronous programming using Scala's Futures. By mixing in the [FutureSupport](http://www.scalatra.org/2.3/api/index.html#org.scalatra.FutureSupport) trait one can easily make their servlet asynchronous. Once this trait is mixed-in into your servlet class, you can return Futures in your `post` and `get` handlers and Scalatra will automagically take care of them. Recently I encountered a minor issue with Scalatra's support for Futures - it is not possible to access `params` or `request` values from code inside a Future. The below code throws a `NullPointerException`.

```scala
get("/someResource/:id") {
  facebookService.signInAsync("someLogin", "somePassword") flatMap { facebookUser ->
    val id = params("id")
    database.getResourceAsync(facebookUser, id)
  }
}
```

Scalatra exposes access to contextual data such as the current user or request parameters via members such as `params` or `request`. These values are implemeted as `DynamicVariables`. Dynamic variables is Scala's feature which allows a `val` to have different values in different scopes. The point is that `DynamicVariable` implementation is based on Java's `ThreadLocal`. Therefore, when executing code in a Future you may not rely on these values since you might be on another thread! An obvious solution to this problem is to retrieve request parameters before entering the Future:

```scala
get("/someResource/:id") {
  val id = params("id")
  facebookService.signInAsync("someLogin", "somePassword") flatMap { facebookUser ->
    database.getResourceAsync(facebookUser, id)
  }
}
```

However, this is not always a very convenient solution. I came up with the following workaround:

```scala
get("/someResource/:id") {
  val currentRequest = request
  facebookService.signInAsync("someLogin", "somePassword") flatMap { facebookUser ->
    withRequest(currentRequest) {
      val id = params("id") 
      database.getResourceAsync(facebookUser, id)
    }
  }
}
```

Firstly, we take a copy of the current request. Later, inside the Future we tell Scalatra to substitute the `request` dynamic variable's value with our copy. Therefore, the call to `params` will use the correct `request` and there will be no error.

### Update

As I recently learned, there is a much better way to solve this issue that is actually built into Scalatra. The way to go is using the AsyncResult  class. Our example would look like this:

```scala
get("/someResource/:id") {
  new AsyncResult { val is =
    facebookService.signInAsync("someLogin", "somePassword") flatMap { facebookUser ->
      val id = params("id") 
      database.getResourceAsync(facebookUser, id)
    }
  }
}
```

AsyncResult  is an abstract class. We create an instance of anonymous type that extends it and overrides is  value. AsyncResult  takes copies of current request  and response  values when created and makes them available to code inside is . You can find more information [here](http://www.scalatra.org/2.4/guides/async/akka.html).
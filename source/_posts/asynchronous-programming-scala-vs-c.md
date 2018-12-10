---
title: 'Asynchronous programming in Scala vs C#'
tags:
  - csharp
  - scala
  - asynchronous programming
url: 60.html
id: 60
categories:
  - .NET
  - Articles
  - Best Of
  - Other topics
date: 2016-03-17 22:33:36
---

[In one of my recent post](http://codewithstyle.info/scalas-option-monad-versus-null-conditional-operator-in-c/) I compared two different approaches that authors of Scala and C# chose to solve the same problem. This post is based on the same idea but the problem being solved is asynchronous programming.

### What's asynchronous programming?

Let me explain by giving you an example. If you have ever used a web framework you might have been wondering how it handles multiple concurrent requests from different users. The traditional approach is to spawn a new thread (or get one from a thread pool) for every request that comes in and release it once the request is served. The problem with this solution is that whenever those threads perform IO operations (such as talking to a database) they simply block and wait for the operation to finish. Therefore, we end up wasting precious CPU time by allowing our threads to be blocked on IO. 

Instead of blocking threads on IO operation we could use an asynchronous database API. Such API is non-blocking. However, running a database query using such an API requires you to provide a **callback**. Callback in this case would be a function that would be invoked once the result is available. So, in the asynchronus model your thread serves the request, runs some computations and when it needs to call the database, it initiates the call and than switches to do some other, useful work. Some other thread will continue execution of your request when the database returns. 

![Asynchronous model example](/images/2016/03/drawit-diagram-1.png "drawit diagram")

### Asynchronous programming in C#

The biggest pain of writing programs in the asynchronous model is the necessity of callbacks. Fortunately, in C# we have lambda functions which allow us to write callbacks with ease. However, even with lambdas we can end up with lot of nesting. The key to asynchronous programming in C# is the **Task** class. Task represents a piece of work that can be either blocking or heavy on processor so it makes sense to run it asynchronously.

```csharp
Task<HttpResponseMessage> getGoogleTask = client.GetAsync("http://google.com");
getGoogleTask.ContinueWith(task => Debug.WriteLine(task.Result));
```

In the first line we create a task that fetches the Google main page. The task starts immedietely on a thread from a default, global thread pool. Therefore, the call itself is not blocking. On the second line we attach a callback which defines what should happen once the result is fetched. As I said, it is easy to introduce nesting with callbacks. What if we wanted to visit Facebook but only if we succeeded fetching the Google page?

```csharp
client.GetAsync("http://google.com").ContinueWith(googleTask =>
{
    if (googleTask.Result.IsSuccessStatusCode)
    {
        client.GetAsync("http://facebook.com").ContinueWith(facebookTask =>
        {
            if (facebookTask.Result.IsSuccessStatusCode)
            {
                Debug.WriteLine("Google and Facebook available!");
            }
        });
    }
});
```

This code isn't very readable. Also, if we wanted to visit more websites, we could end up with even more levels of nesting. C# 5.0 introduced an excellent language feature that lets you write asynchronous code just as if it was synchronous: the **async** and **await** keywords. The above example can be rewritten as follows:

```csharp
var googleResponse = await client.GetAsync("http://google.com");
if (googleResponse.IsSuccessStatusCode)
{
    var facebookResponse = await client.GetAsync("http://facebook.com");
    if (facebookResponse.IsSuccessStatusCode)
    {
        Debug.WriteLine("Google and Facebook available!");
    }
}
```

One caveat about async/await is that the method containing any **await** calls must itself be declared as **async**. Also, the return type of such a method must be a Task. Therefore, the asynchronous-ness always propagates upstream. This actually makes sense - otherwise you would need to synchronously wait for a task to finish at some point. Modern web frameworks such as ASP.NET MVC let you declare the methods that handle the incoming requests as asynchronous.

```csharp
public async Task<ActionResult> FetchWebsites()
{
    HttpClient client = new HttpClient();
    var googleResponse = await client.GetAsync("http://google.com");
    if (googleResponse.IsSuccessStatusCode)
    {
        var facebookResponse = await client.GetAsync("http://facebook.com");
        if (facebookResponse.IsSuccessStatusCode)
        {
            ViewBag.Message = "Google and Facebook available!";
        }
    }
    
    return View();
}
```

One more thing about C# tasks - with them executing stuff in parallel is incredibly easy.

```csharp
string[] websites = new string[] { "http://google.com", "http://facebook.com" };
Task<HttpResponseMessage>[] tasks = websites.Select(website => client.GetAsync(website)).ToArray();
HttpResponseMessage[] responses = await Task.WhenAll(tasks);
```

**Task.WhenAll** creates a task that will be finished when all tasks from the provided array are finished.

### Asynchronous programming in Scala

Let's have a look at how Scala approaches the problem. One of the approaches to asynchronous programming is to use [Futures](http://docs.scala-lang.org/overviews/core/futures.html). Future is a class that has very similiar semantics to C#'s Task. Unfortunately, there is no built-in asynchronous HTTP client in Scala, but let's assume we've got one and it's interface looks like this:

```scala
trait HttpClient {
    def get(url: String): Future[Response]
    case class Response(content: String, ok: Boolean)
}
```

We can write code that looks very similiar to the C# example with **flatMap**:

```scala
client.get("http://google.com").flatMap(googleResp =>
  if (googleResp.ok) {
    client.get("http://facebook.com").map(facebookResp => {
      if (facebookResp.ok) {
        Console.printf("Google and Facebook are on-line!")
      }
    })
  } else Future.successful()
)
```

Flatmap invoked on a future takes a callback that will be execute once the result of that future is available. Since that callback must return a Future itself, we must return an empty future (Future.successful) in the else branch of our if. When fetching the Facebook page, we use map instead of flatMap because we don't want to start another future inside the callback. Again, the main issue with this code is that it is nested. Very similarly to [how Scala handles nested null checks with Option monad](http://codewithstyle.info/scalas-option-monad-versus-null-conditional-operator-in-c/), here we can again use the for-comprehension syntax to get rid of nesting!

```scala
for {
  googleResp <- client.get("http://google.com") if googleResp.ok
  facebookResp <- client.get("http://facebook.com") if facebookResp.ok
} yield Console.printf("Google and Facebook are on-line!")
```

As you might have expected, parallel processing is also supported with Futures:

```scala
val websites = List("http://google.com", "http://facebook.com")
val responses = Future.sequence(websites.map(client.get))
```

An example of a web framework that supports asynchronous request handlers is [Scalatra](http://www.scalatra.org).

### Conclusions

As you can see, C# and Scala approach asynchronous programming similliarly. What I find interesting here is how Scala handles callback nesting with the generic mechanism of for comprehension and C# introduces a separate language feature for that. This is exactly the same pattern as in [Option monad vs null-conditional operator](http://codewithstyle.info/scalas-option-monad-versus-null-conditional-operator-in-c/). To be honest, I find the async/await overall a bit more awesome - it really makes you feel as if you were writing synchronous code. 

**Update:** as pointed out by Darren and Yann in comments, you can also do async/await in Scala thanks to [this library](https://github.com/scala/async). There is also a [pending proposal](http://docs.scala-lang.org/sips/pending/async.html) to add it to the language that admits that it's inspired by C#'s asyns/await syntax.
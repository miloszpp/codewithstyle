---
title: 'C# in Depth: book notes'
tags:
  - book
  - csharp
url: 133.html
id: 133
categories:
  - .NET
  - Articles
  - Other topics
  - Thoughts
date: 2017-01-15 20:03:55
---

![pobrane](/images/2017/01/pobrane-1.jpeg)

I just finished reading this must-read position for C# developers. I believe that it’s very easy to learn a programming language to an extent that is sufficient for creating software. Because of that, one can easily lose motivation to dig deeper and gain better understanding of the language. C# in Depth is a proof of why one shouldn’t stop at this point. There is a lot to learn by looking at the details of a language, how it evolved and how some of it’s features are implemented. I think the book is fantastic. I loved the author’s writing style which is very precise (very little hand waving) but not boring at the same time. It feels that he’s giving you just the right amount of detail. Here are a couple of interesting things I learned about when reading the book. The list is by no means complete but it gives a taste of what’s in the book.

*   I learned that it’s possible to support LINQ query expressions for your own types very easily. The mechanism is convention-based - there is no specific interface to implement. Your type must have methods that match some specific signatures. This didn't sound well to me in the first place, but if you think about it, it allows for greater flexibility. For example, with such approach you can add query expression support to existing types (which you don’t have control over) with extension methods.
*   I finally understood why the keywords used to indicate variance in generic types are called out and in. Generic type parameter can be covariant if it’s used for values that are coming out of an API (something’s coming out so you can only increase the restriction on it when deriving). Conversely, when value is an input of an API it’s type can be contravariant (something’s coming in, so you can relax the restrictions when deriving). This explanation plays well with my intuition of how collections can be covariant as long as they are immutable (i.e. there are no inputs to the API)
*   I understood how dynamic typing is implemented in C# and how to create your own types which can react dynamically (with IDynamicMetaObjectProvider, DynamicObject and ExpandObject). The chapter explaining what code is generated when making dynamic calls is the most complex (and most interesting) piece of the book.
*   I understood what code is generated when using the async/await feature and what are the consequences. For example, the code inside an async method does not execute until you await it. Therefore, argument validation wouldn’t give immediate feedback to the caller unless the method is awaited at the point of calling. The same applies to iterators.
*   I learned that something as simple as a foreach loop is actually doing a lot of work under the hood - it creates a try/catch/finally block and disposes of the enumerator if it happens to implement Disposable.
*   I embraced the complexity of type inference of lambda expression parameters and return types.

To sum up, I totally recommend reading this book. It’s not a small time investment, but I think it’s totally worth it.
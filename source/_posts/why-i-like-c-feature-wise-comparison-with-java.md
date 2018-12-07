---
title: 'Why I like C#: feature-wise comparison with Java'
tags:
  - 'c#'
  - java
url: 10.html
id: 10
categories:
  - .NET
  - Articles
  - Other topics
  - Thoughts
date: 2015-11-22 15:30:00
---

Recently I was browsing Quora and was quite surprised to stumble upon this question: [Java vs C#. Is Microsoft finally closing the gap?](https://www.quora.com/Java-vs-C-Is-Microsoft-finally-closing-the-gap) I decided to have a closer look and found more of similiar questions there. Furthermore, at the place where I am currently working at, I am the only person with .NET background amongst mostly JVM people. We are all working on Scala projects and my colleagues are often surprised when I tell them that this or that Scala feature is also available in C#. This makes me want to write a blog post about how cool a language C# is, especially when compared with Java. I want to underline that I'm speaking only about the language features and **not** about things like popularity, cross-platformness, ability to deploy easily, etc.

Generics
--------

C# creators were in a great situation since they could learn from Java's mistakes. They didn't waste the opportunity and did the right thing. The main problem with Java's generics is **type erasure**. The term means that the information about the type parameter of a generic type is not available at runtime. In simple words, this:

List<string> list = new LinkedList<string>();

...becomes this:

List list = new LinkedList();

Type erasure makes writing generic types more difficult and less clean. For example, sometimes generic methods have to explicitly take a `Class` object representing the type parameter (like [here](http://stackoverflow.com/questions/3437897/how-to-get-a-class-instance-of-generics-type-t)). In C# this is not the case. You can easily access the type of the type parameter:

class List<T> 
{
    public void PrintType() {
        Console.WriteLine(typeof(T));
    }
}

 

Lambdas, higher-order functions and LINQ
----------------------------------------

Not long ago I found [this article](https://github.com/winterbe/java8-tutorial) on [Hacker News](https://news.ycombinator.com/). It discusses some of the new features of Java 8 such as lambdas, streams and functional interfaces. These things are called _modern_ Java whereas in C# they have been available for quite a long time (not to mention that they have been available in Haskell or Ocaml for even longer). While not everyone has to agree about superiority of functional over imperative programming, it's hard to disagree that processing collections with higher-order functions (such as [map](https://github.com/winterbe/java8-tutorial#map)/[select](https://msdn.microsoft.com/pl-pl/library/bb548891(v=vs.110).aspx) or [filter](https://github.com/winterbe/java8-tutorial#filter)/[where](https://msdn.microsoft.com/library/bb534803(v=vs.100).aspx)) is cleaner, less error-prone and much more readable than doing it with loops. Even though Java has already adopted lambdas and higher-order functions, it seems that C# has better support for them. Examples?

*   In Java 8, you need to convert collections to `Stream` before calling `map` or `filter`
*   C# has built-in syntactic sugar for such opearations which makes such code even more readable and cleaner

 

Type inference
--------------

Type inference is a nice feature that allows you not to declare the type of a variable if it's being initialized on the same line. While it's not as great as in Scala or Haskell, it certainly lets you cut some boilerplate code. [Java does also have some type inference](https://docs.oracle.com/javase/tutorial/java/generics/genTypeInference.html) but it is limited to generic methods. With type inference, the below declaration:

Dictionary<int, List<Tuple<int, int>>> graph = new Dictionary<int, List<Tuple<int, int>>>();

...can be written as:

var graph = new Dictionary<int, List<Tuple<int, int>>>();

 

Asynchronous code
-----------------

C# 5.0 introduced excellent support for asynchronous programming. The `async` and `await` keywords let you replace callback-style programming with code that looks exactly as if it were synchronous. It makes the code much cleaner and far easier to read. The comparison with Java is especially striking if you look at pre-Java 8 code where in order to execute a piece of code asynchronously, you had to create an anounymous type with one method! Have a look at usage of the [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client) library:

AsyncHttpClient asyncHttpClient = new AsyncHttpClient();
Future<integer> f = asyncHttpClient.prepareGet("http://www.ning.com/").execute(
   new AsyncCompletionHandler<integer>(){

    @Override
    public Integer onCompleted(Response response) throws Exception{
        // Do something with the Response
        return response.getStatusCode();
    }

    @Override
    public void onThrowable(Throwable t){
        // Something wrong happened.
    }
});

...and compare it with this C# code:

async Task<int> AccessTheWebAsync()
{ 
    HttpClient client = new HttpClient();
    Task<string> getStringTask = client.GetStringAsync("http://msdn.microsoft.com");
    DoIndependentWork();
    string urlContents = await getStringTask;
    return urlContents.Length;
}

 

Value types
-----------

Value types is part of the reason why there is a _C_ in _C#_. There are two kinds of types in C# - value types and reference types. Value types differ from reference types mainly in the assignment sementics. When you assign a reference to a new variable, this variable points to the same object. When you assign a value type to a new variable, the whole piece of memory holding the data in the type is copied. This is great for lightweight objects representing data. In some situations it might save you from writing the `equals` and `hashCode` operators. What's more, value types cannot be null which makes them safer than reference types. Finally, value types make primitive types such as `int` or `double` more natural. In Java, every type is a reference type.

Extension methods
-----------------

Extension methods allow you to add functionality to an object (even if it had already been compiled). One of the cool uses of extension methods is providing a concrete method for an interface. Also, they allow better code reuse and makes it easier to write fluent APIs. Example of an extension method:

public interface Animal {
    int Legs { get; }
}

public static class AnimalExtensions {
    public static void PrintDescription(this Animal animal) {
        Console.WriteLine("I have {0} legs", animal.Legs);
    }
}

animal.PrintDescription();

 

C# 6.0 features
---------------

Finally, there are many great features introduced in C# 6.0. The language seems to be gravitating towards functional programming, which I think is a good idea, but most of them do not require the programmer to learn a new paradigm. To name some of the most exciting features of C# 6.0:

*   Expression bodied methods - syntax improvment which makes you write shorter code
*   Conditional null operator - which allows writing safer code (which feels like simplified `Maybe` monad)
*   Expression filters - convenient syntax for exception handling
*   Using static members - again, an improvment to make your code even shorter

 

Conclusion
----------

I have named just a few of the language features of C# which I believe make it a superior language to Java. Obviously, there are many more things to look at when choosing a language than its features. However, I think it's worth mentioning that thanks to Mono, Xamarin and Microsoft's BizSpark program for startups, .NET became much more accessible to small companies and startups than it was a decade ago.
---
title: Understand monads with LINQ
tags:
  - 'c#'
  - functional programming
url: 167.html
id: 167
categories:
  - .NET
  - Best Of
  - Other topics
  - Tutorials
date: 2017-03-18 12:19:35
---

This post is another attempt on explaining **the M word** in an approachable way. This explanation will best suite C# developers who are familiar with LINQ and query expressions. However, if you are not familiar with C# but would like to learn how powerful and expressive some of its features are, please read on!

### Recap of LINQ and query expressions

LINQ is a technology introduced in C# 3.0 and .NET 3.5. One of its major applications is processing collections in an elegant, declarative way. Here's an example of LINQ's s_elect_ expression:

var numbers = new\[\] { 1, 2, 3, 4, 5 };
var squares = numbers.Select(x => x * x);

Query expressions are one of the language features which constitute LINQ. Thanks to it LINQ expressions can look in a way which resembles SQL expressions:

var squares = from x in numbers select x * x;

Before LINQ you would need to write a horrible, imperative loop which literates over the _numbers_ array and appends the results to a new array.

### Single element collection: Maybe class

It's pretty easy to understand what _select_ expression does in the above example: it apples a given expression to each element of a collection and produces a collection containing the results. Let's now imagine that instead of arbitrary collection, we are working with a special kind of collection - one that can have either one element or no elements at all. In other words, it's either empty, or full. How should _select_ expression act on such a collection? Exactly the same way that it works with regular collections. If our collection has one element than apply the given expression to it and return a new collection with the result. If the collection is empty, just return an empty collection. Note that such a special collection is actually quite interesting - it represents an object that either has a value or is empty. Let's create such an object and call it _Maybe_.

public class Maybe<TValue>
{
    private readonly TValue value;
    private readonly bool hasValue;
        
    internal Maybe(TValue value, bool hasValue)
    {
        this.value = value;
        this.hasValue = hasValue;
    }
}

Let's create two factory methods to allow more convenient creation of instances of _Maybe_.

public static class MaybeFactory
{
    public static Maybe<T> Some<T>(T value) => new Maybe<T>(value, true);

    public static Maybe<T> None<T>() => new Maybe<T>(default(T), false);
}

Thanks to type inference in generic method calls and the **static using**feature we can now simply write:

var some = Some(10);
var none = None<int>();

### Making Maybe LINQ-friendly

Since we've already discussed how _select_ would work on _Maybe_, let's implement it! Adding support for query expressions to your custom types is surprisingly easy. You just need to define a method which confirms to a specific signature (it's an interesting design decision by C# creators which allows more flexibility than requiring the type to implement a specific interface).

public Maybe<TResult> Select<TResult>(Func<TValue, TResult> mapperExpression)
{
    if (this.hasValue)
    {
        return MaybeFactory.Some(mapperExpression(this.value));
    }
    return MaybeFactory.None<TResult>();
}

What's going on here? Firstly, let's take a look at the signature. Our method takes a function which transforms the value contained by _Maybe_ to another type.  It returns an instance of _Maybe_ containing an instance of the result type. If it's confusing, just replace _Maybe_ with _List_ or _IEnumerable. _It makes perfect sense to write a _select_ expression which transforms a list of _ints_ to a list of _strings_. It works the same way with our _Maybe_ type. ![](http://codewithstyle.info/wp-content/uploads/2017/03/select-1.png "select") Now, the implementation. There are two cases:

*   If the object contains a value than apply the _mapper_ function and return a new _Maybe_ instance with the result
*   If the object is empty, there is nothing to convert - return a new empty _Maybe_ instance

Let's give it a try:

Maybe<int> age = Some(27);
Maybe<string> result = from x in age select string.Format("I'am {0} years old", x);

Nice! We can now use _select_ expressions with _Maybe_ type.

### Taking it one step further

Let's now imagine that given an employee's id, our goal is to return the name of theirs supervisor's supervisor. A person can but does not have to have a supervisor. We are given a repository class with the following method:

public Person GetPersionById(Guid id) { ... }

And a _Person_ class:

class Person
{
    public string Name { get; set; }
    public Person ReportsTo { get; set; }
}

In order to find the person's supervisor's supervisor's name we would need to write a series of _if_ statements:

public static string GetSupervisorSupervisorName(Person employee)
{
    if (employee != null)
    {
        if (employee.ReportsTo != null)
        {
            if (employee.ReportsTo.ReportsTo != null)
            {
                return employee.ReportsTo.ReportsTo.Name;
            }
        }
    }

    return null;
}

Can we improve this code with our new _Maybe_ type? Of course we can! First of all, since _Maybe_ represents a value which may or may not exist, it seems reasonable for _GetPersonById_ to return _Maybe<Person>_ instead of _Person_.

public static Maybe<Person> GetPersonById(Guid id)

Next, let's modify the _Person_ class. Since a person can either have or not have a supervisor, it's again a good fit for the _Maybe_ type:

class MonadicPerson
{
    public string Name { get; set; }
    public Maybe<MonadicPerson> ReportsTo { get; set; }
}

Given these modifications we can now rewrite _GetSupervisorSupervisorName_ in a neater and more elegant way:

public static Maybe<string> GetSupervisorName(Maybe<MonadicPerson> maybeEmployee)
{
    return from employee in maybeEmployee
           from supervisor in employee.ReportsTo
           from supervisorSupervisor in supervisor.ReportsTo
           select supervisorSupervisor.Name;
}

Why is this better than the previous version? First of all, we explicitly represent the fact that given a person, the method might or might not return a valid result. Previously, the method always returned a _string_. There was no way to indicate that it can sometimes return _null_ (apart from a comment). A user of such a method could forget to perform null check and in consequence be surprised by a runtime error. What's more, we avoid the nesting of if statements. In this example we only go two levels deep. What if there were 5 levels? Code without these nested if statements is much cleaner and more readable. It expresses the actual logic, not on the boilerplate of null-checking.

### Making it work

If you're copying these snippets to Visual Studio, you might have noticed that the last one won't compile. By implementing _Select_ we told the compiler how to apply functions to values inside _Maybe_ instances. However, here we have a slightly more complex situation. We take a value which sits inside a _Maybe_ instance and apply a function to it. As a result we get another _Maybe_ instance, so now we have a _Maybe_ inside a _Maybe_. The compiler doesn't know how to handle this situation and we need to tell it by implementing _SelectMany_.

public Maybe<TResult> SelectMany<TIntermediate, TResult>(
    Func<TValue, Maybe<TIntermediate>> mapper,
    Func<TValue, TIntermediate, TResult> getResult
)
{
    if (this.hasValue)
    {
        var intermediate = mapper(this.value);
        if (intermediate.hasValue)
        {
            return MaybeFactory.Some(getResult(this.value, intermediate.value));
        }
    }
    return MaybeFactory.None<TResult>();
}

The first parameter to _SelectMany_ is a function which takes a value (which sits inside _Maybe_) and returns a new _Maybe_. In our example, that would be a function which takes a _Person_ and returns its _ReportsTo_ property. The second parameter is a function which takes the original value, the value sitting inside _Maybe_ returned by the first parameter and transforms both into a result. In our case that would be a function which takes a _Person_ and returns its _Name_. Inside the implementation we have the nested if statements that we had to write when we didn't use the _Maybe _type. And this is the crucial idea about monads - they help you hide ugly boilerplate code and let the developer focus on the actual logic. Again, let me draw a diagram for those of you who prefer visual aids: ![](http://codewithstyle.info/wp-content/uploads/2017/03/SelectMany.png "SelectMany")

### Ok, so what's exactly a Monad?

Monad is any generic type which implements _SelectMany _(strictly speaking, this is far from a formal definition, but I think it's sufficient in this context and captures the core idea). _SelectMany_ is a slightly more general version of an operation which in the functional programming world is referred to as **bind**. Monadic types are like wrappers around some values. Binding monads is all about composing them. By wrapping and unwrapping of the values inside monads, we can perform additional operations (such as handling empty results in our case) and hide them away from the user. _Maybe_ is a classic example of a monad. Another great candidate for monad is C#'s _Task<T>_ type. You can think of it as a type that wraps some value (the one that will be returned when the task completes). By combining tasks you describe that one task should be executed after the other finishes.

### Summary

I hope this article helped you understand what monads are about. If you find this interesting, check out the F# programming language where monads are much more common and feel more natural. Check out this excellent resource about F#: https://fsharpforfunandprofit.com/. It's also worth mentioning that there exists an interesting C# library which exploits the concepts I described in this article: https://github.com/louthy/csharp-monad. Check it out if you're interested.
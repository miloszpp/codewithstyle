---
title: 'Scala''s Option monad versus null-conditional operator in C#'
tags:
  - functional programming
  - scala
url: 6.html
id: 6
categories:
  - .NET
  - Articles
  - Best Of
  - Other topics
date: 2016-02-11 21:21:00
---

Today I will talk about an awesome feature of C# 6.0. We will see how it can help us understand monads in Scala!

Null-conditional operator
=========================

Imagine we have a nested data model and want to call some method on a property nested deeply inside an object graph. Let's assume that Article does not have to have an Author, the Author does not have to have an Address and the address does not have to have a City (for example this data can be missing from our database).

public class Address {
 public string Street { get; set; }
 public string City { get; set; } // can be null
}

public class Author {
 public string Name { get; set; }
 public string Email { get; set; }
 public Address Address { get; set; } // can be null
}

public class Article {
 public string Title { get; set; }
 public string Content { get; set; }
 public Author Author { get; set;} // can be null
}

Console.WriteLine(article.Author.Address.City.ToUpper());

This is very unsafe code since we are at risk of `NullReferenceException`. We have to introduce some null checks in order to avoid the exception.

if (article != null) {
 if (article.Author != null) {
  if (article.Author.Address != null) {
   if (article.Author.Address.City != null) {
    Console.WriteLine(article.Author.Address.City.ToUpper());
   }
  }
 }
}

Yuck! So much boilerplater code to do a very simple thing. It's really unreadable and confusing. Fortunately, C# 6.0 introduces the [null-conditional operator](https://msdn.microsoft.com/en-us/library/dn986595.aspx). The new operator denotes `?.` and can be used instead of the regular `.` whenever it is possible that the value on the left can be `null`. For example, the below piece can be read as "call `ToUpper` only if `bob` is not `null`; otherwise, just set `bobUpper` to `null`".

var bob = "Bob";
var bobUpper = bob?.ToUpper();

Returning to our previous example, we can now safely write:

Console.WriteLine(article?.Author?.Address?.City?.ToUpper());

 

The `Option` type
=================

As I explained in one of my previous posts, in Scala we avoid having `null` variables at all cost. However, we would still like to be able to somehow reflect the fact that a piece of data is optional. The `Option[T]` type can be used to explicitly mark a value as optional. For example, vale `bob` with type `Option[String]` means that `bob` can either hold a `String` value or nothing:

val someBob: Option\[String\] = Some("Bob")
val noBob: Option\[String\] = None

Therefore, we can easily model the situation from the previous example as follows:

case class Address(street: String, city: Option\[String\])
case class Author(name: String, email: String, address: Option\[Address\])
case class Article(title: String, content: String, author: Option\[Author\])

Notice how, compared to C#, Scala forces us to explicitly declare which field is and which field is not optional. Now, let's look at how we could implement printing article's author's city in lower case:

if (article.author.isDefined) {
  if (article.author.get.address.isDefined) {
    if (article.author.get.address.get.city.isDefined) {
      println(article.author.get.address.get.city.get)
    }
  }
}

This naive approach is not a big improvement when compared to the C# version. However, Scala lets us do this much better:

for {
  author <- article.author
  address <- author.address
  city <- address.city
} yield println(city.toLowerCase())

Although this version is not as short as the one with C#'s null-conditional operator, it's important that we got rid of the boilerplate nested `if` statements. What remained is a much more readable piece of code. This is an example of the for-comprehension syntax together with the monadic aspect of the `Option` type.

The `Option` monad
==================

Before I exaplain what exactly is going on in the above piece of code, let me talk more about methods of the `Option` type. Do you remember the `map` method of the `List` type? It took a function and applied it to every element of the list. Interestingly, `Option` does also have the `map` method. Think of `Option` as of a `List` that can have one (`Some`) or zero (`None`) elements. So, `Option.map` takes a function and if there is a value inside the `Option`, it applies the function to the value. If there is no value inside the `Option`, `map` will simply return `None`.

scala> val address = Address("street", Some("New York"))
address: HelloScala.Address = Address(street,Some(New York))

scala> address.city.map(city => city.toLowerCase())
res1: Option\[String\] = Some(new york)

scala> val address = Address("street", None)
address: HelloScala.Address = Address(street,None)

scala> address.city.map(city => city.toLowerCase())
res2: Option\[String\] = None

Now, can we somehow use it with our initial problem? Let's see:

val cityLowerCase = article.author.map { author =>
  author.address.map { address =>
    address.city.map(city => city.toLowerCase)
  }
}

I think it looks slightly better than the nested if approach. The problem with this is that the type of `cityLowerCase` is `Option[Option[Option[String]]]`. The actual result is deeply nested. What we would prefer to have is an `Option[String]`. There is a method similiar to `map` which would give us exactly what we want - it's called `flatMap`.

val cityLowerCase: Option\[String\] = article.author.flatMap { author =>
  author.address.flatMap { address =>
    address.city.map(city => city.toLowerCase)
  }
}

`Option.flatMap` takes a function that transforms an element inside the option to another option and returns the result of the transformation (which is a non-nested option). The equivalent for `List` is `List.flatMap` which takes a function that maps each element of the list to another list. At the end, it concatenates all of the returned lists.

scala> List(1, 2, 3, 4).flatMap(el => List(el, el + 1))
res3: List\[Int\] = List(1, 2, 2, 3, 3, 4, 4, 5)

The fact that `Option[T]` and `List[T]` have the `flatMap` means that they can be easily composed. In Scala, every type with the `flatMap` method is a monad! In other words, a monad is any generic type with a type parameter which can be composed with other instances of this type (using the `flatMap` method). Now, back to for-comprehension. The nice syntax which allows us to avoid nesting in code is actually nothing more than a syntactic sugar for `flatMap` and `map`. This:

val city = for {
  author <- article.author
  address <- author.address
  city <- address.city
} yield println(city.toLowerCase())

...translates into this:

val cityLowerCase: Option\[String\] = article.author.flatMap { author =>
  author.address.flatMap { address =>
    address.city.map(city => city.toLowerCase)
  }
}

For comprehension works with any monad! Let's look at an example with lists:

scala> for {
  el <- List(1, 2, 3, 4)
  list <- 1 to el
} yield list
res4: List\[Int\] = List(1, 1, 2, 1, 2, 3, 1, 2, 3, 4)

For each element in the first list we produce a list ranging from 1 to this element. At the end, we concatenate all of the resulting lists.

Conclusion
==========

My main point here is to show that both C# and Scala introduce some language elements to deal with deep nesting. C# has null-conditional operators which deal with nesting null checks inside if statements. Scala has a much more generic mechanism which allows to avoid nesting with for-comprehension and `flatMap`. In the next post I will compare C#'s `async` keyword with Scala's `Future` monad to show the similarities in how both languages approach the problem of nested code.
---
title: 'Scala for C# developers - part III'
tags:
  - csharp
  - scala
url: 9.html
id: 9
categories:
  - Other topics
  - Scala
  - Tutorials
date: 2016-01-03 22:20:00
---

I'm back from a rather lenghty break and would like to continue the **Scala for C# developers** series. So far I have covered the syntax, the basics of OO in Scala and functions. In this post I will look at the `Option` type and pattern matching.

Issues with `null` references
-----------------------------

If you have programmed in C# (or Java, or any other language that supports `null` references) you must already know the pain of `NullReferenceException`. This exception is thrown whenever you are expecting that a variable points to an actual object but in reality it does not point to anything. Therefore, calling a method on such reference would result in the exception. There is a famous quote from Tony Hoare who introduced the concept of `null` references claiming that it was his billion-dollar mistake: _I call it my billion-dollar mistake. It was the invention of the null reference in 1965. At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W). My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years._

The `Option` type
-----------------

What does it mean when a `NullReferenceException` is thrown? As I said, it means that the CLR was expecting a reference to an object but found an empty refence and does not know what to do with it. In the majority of cases, it means that you as the programmer should have thought about it and check for the `null` reference before doing anything with it. Unfortunately, it would require some great discipline to keep track of all references that could become `null` and to take care of each and every one of them. The `Option` type comes to rescue. The idea is to force the compiler to do the hard work for you. `Option[T]` is an abstract type which has two subclasses: `Some[T]` and `None`. For example, a value of type `Option[Int]` represents an object that can, but does not have to hold some integer. If this `Option` is an instance of `Some` than the object has some value. If it's `None` than it does not have any value. So, `None` is like `null` except we explicitly declare that an object can be `None` by making it an `Option`. If we decide to use `Option` types in our project we must forget about `null` references completly. Therefore, whenever we expect a value of type `T` to be optional, we must declare it as `Option[T]`. Thanks to that, the compiler will forbid us from writing such code:

def makeUpper(text: Option\[String\]) = text.toUpperCase()

For this to compile, we must explicitly handle the case when the provided argument is undefined.

def makeUpper(textOpt: Option\[String\]) =
    textOpt match {
        case Some(text) => text.toUpperCase
        case None => ""
    }

Such code, although lengthier, is much, much safer than traditional code which allows use of `null` references. Of course, the key thing is to make sure that there is never a `null` inside `Some` value. However, this is easy to ensure as long as we decide not to use `null` references in the whole project.

Pattern matching
----------------

The above code snippet introduces some new syntax. The `match` construct is the Scala syntax for **pattern matching**. It is a very powerful tool common in functional programming. You can think of it as a much more advanced `switch` statement which always returns a value as a whole. In the above example, the value of `textOpt` is examined. It is an instance of `Option` type and we know that it has two subclasses. Therefore, there are two `case` branches. The first branch demonstrates how the value contained inside `Some[T]` can be extracted. Pattern matching can be used with simple types:

x match {
          case 1 => println("1")
          case 2 => println("2")
          case _ => println("other value")
      }

Additionally, pattern matching works very well with case classes which we discussed in the previous post.

abstract class Animal
case class Dog(name: String) extends Animal
case class Fish(kind: String) extends Animal

animal match {
   case Dog(name) => println("This is a dog named " + name)
   case Fish(kind) => println("This is a fish of kind " + kind)
}
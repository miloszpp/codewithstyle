---
title: 'Scala for C# developers - part II'
tags:
  - csharp
  - scala
url: 11.html
id: 11
categories:
  - Other topics
  - Scala
  - Tutorials
date: 2015-11-13 00:08:00
---

This is the second post in the series. Click [here](http://wordpress1653421.home.pl/home/platne/serwer16812/public_html/codewithstyle/?p=12) to see the previous part. In the previous post I covered the basics of Scala syntax as well as some comparison of OOP in Scala and C#. Today, I will focus on lambdas and higher-order functions.

Functions as function parameters
--------------------------------

You are most likely familiar with lambda expessions in C#. Lambda expression is simply an anonymous function. Lambdas are useful when you want to pass a piece of code as a parameter to some other function. This concept is actually one of the cornerstones of functional programming. One great example of how useful lambdas are operations on collections. The following piece of code takes a list of integeres, filters out odd numbers and multiplies the remaining numbers by 5.

var list = new List<int> { 1, 2, 3, 4, 5 };
var multiplied = list.Where(x => x % 2 == 0).Select(x => x * 5);

In Scala, it would look surprisingly similiar:

val list = List(1, 2, 3, 4, 5)
val multiplied = list.filter(x => x % 2 == 0).map(x => x * 5)

Scala uses more traditional FP names for `map` and `filter` but apart from this, the code looks very similiar. In Scala, we can make it a bit tighter (and less readable):

val list = List(1, 2, 3, 4, 5)
val multiplied = list.filter(_ % 2 == 0).map(_ * 5)

As you can see, Scala allows you to use anonymous parameters inside anonymous functions. However, be careful when using the underscore notation. The `(_ * 5) + _` expression **does not** translate into `x => (x * 5) + x`. Instead, the second underscore is assumed to be the second anonymous parameter of the lambda, therefore meaning this: `(x, y) => (x * 5) + y`.

Returning functions
-------------------

C# not only allows to have functions which take functions as parameters but also functions that return other functions. In the following piece, the `GetMultiplier` function takes a single integer and returns a function that can multiply it by any other integer.

static Func<int, int> GetMultiplier(int a) {
    return x => a * x;
}
var multiplier = GetMultiplier(5);
var multiplied = list.Select(multiplier);

Let's see how would it look in Scala:

def getMultiplier(x: Int): Function1\[Int, Int\] = {
    y: Int => x * y
}      
val multiplier = getMultiplier(5)
val multiplied2 = list.map(multiplier)

Again, it looks fairly similiar. The `Function1[Int, Int]` has the same semantics as Func%lt;int, int> - it represents a one-argument function that takes an integer and returns an integer. Interestingly, in Scala `Function1[Int, Int]` can be denoted as `Int => Int`.

def getMultiplier(x: Int): Int => Int = {
    y: Int => x * y
}

We can go one step further and rewrite the above function as:

def getMultiplier(x: Int)(y: Int) = x * y

This certainly looks odd - our function now has two parameter lists! It is Scala's special syntax for functions returning functions. You can pass one integer to `getMultiplier` and what you get is a **partially applied** function. What is the type of `getMultiplier` now? It's `Int => (Int => Int)` which can also be written simply as `Int => Int => Int`. This technique is called **currying**. The idea of currying is that a function with multiple parameters can be treated as a function that takes the first parameter and returns a function that takes a second parameters which returns a function... and so on.
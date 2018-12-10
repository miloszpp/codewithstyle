---
title: 'Scala for C# developers - part I'
tags:
  - csharp
  - scala
url: 12.html
id: 12
categories:
  - Other topics
  - Scala
  - Tutorials
date: 2015-10-24 10:59:00
---

Recently, after three years of focusing mainly on the .NET platform, I've changed jobs. My current company uses Scala for server-side programming in their projects. I was very happy for this transition. Both Scala and C# can be considered hybrid functional and object-oriented programming languages. However, Scala seemed to feel more functional than C# - more built-in functional constructs, tighter syntax, default immutability, etc. While this is true, I was surprised how many similarities these languages. I concluded that as long as you have already seen the more functional side of C#, it is really easy to transition to Scala. This post series will discuss some of the similarities and differences between Scala and C#.

Syntax
------

The syntax in Scala is indeed quite different from C# syntax. Let's have a look at this `HelloWorld` program in Scala.

```scala
object HelloWorld {
    def main(args: Array[String]): Unit = {
      println("Hello, world!")
    }
  }
```

First of all, the `object` keyword seems unfamiliar. In Scala, singleton objects are part of the language. It is like declaring a class and saying that there can be only one instance of this class - and this instance is like a global variable, accessible by name of the class. The concept is not very similiar to **static classes** in C#. Another difference is method declaration. As you can see, in Scala the type (of method or variable) comes **after** the name, not before. The `def` keyword marks a method declaration. The `Unit` type is a bit like `void` in C# - it is the return type of a method which does not return any sensible value. One more thing - the `println` call is not preceeded with any class/object name. In Scala, `objects` can behave like `namespaces` in C#. It is possible to import all methods from an object. The [using static members](https://roslyn.codeplex.com/wikipage?title=Language%20Feature%20Status&referringTitle=Documentation) feature in C# 6.0 gives you the same behaviour. It is possible to write the above piece in a more compact way:

```scala
object HelloWorld {
   def main(args: Array[String]) = println("Hello, world!")
}
```

As you can see, the braces can be omited for single-line methods. Also, the `Unit` type disappeared - now it is inferred by the compiler (similarly to how the `var` keyword works in C#). Again, C# 6.0 brings us something similiar - the **expression-bodied members**.

Classes and objects
-------------------

I have already introduced the `object` keyword. Let's now have a look at regular classes.

```csharp
class Ship(name: String, x: Double, y: Double) {
    val positionX = x
    val positionY = y
    def distanceFrom(ship: Ship): Double = Math.sqrt(
            Math.pow(ship.positionX - positionX, 2) + 
            Math.pow(ship.positionY - positionY, 2))
}
```

```scala
object HelloWorld {
   def main(args: Array[String]): Unit = {
      println("Hello, world!")
      println(new Ship("Endevour", 3, 4).distanceFrom(new Ship ("Falcon Millenium", 0, 0)))
   }
}
```

Again, let's look at the differences, case by case. There is no such thing as class constructor here, as we know it from C#. The constructor arguments are writen next to the class name. The initialization code lies directly in the class body. You could conclued that the class can only have one construcor in Scala. This is not true - additional constructors can be provided as **secondary constructors**. In the following lines there are two field declarations. Fields are public by default. The keyword used here is `val` which means that `x` and `y` are **immutable**. Immutability is at the heart of functional programming. Immutable values are values that cannot be modified. It may seem counterintuitive at first but in fact immutable values can help you eliminate whole classes of errors form your programs. I will discuss immutability in more detail in one of the future posts. For now, I recommend [this article](http://fsharpforfunandprofit.com/posts/correctness-immutability/). Member fields do not have to have type declarations - the compilers infers the correct types. The `distanceFrom` method declaration is pretty straightforward. You may notice that there is not `return` statement here. This is because in Scala the method always, by default, returns the last expression in its body. In our case, there is only one expression. Class instantiation is very C#-like - we use the `new` keyword and provide constructor arguments.

### Case classes

Scala introduces a very useful concept called **case class**. Let's see how we could rewrite the above code in a more succinct way.

```scala
case class Ship(name: String, positionX: Double, positionY: Double) {
    def distanceFrom(ship: Ship): Double = Math.sqrt(
            Math.pow(ship.positionX - positionX, 2) + 
            Math.pow(ship.positionY - positionY, 2))
}

object HelloWorld {
   def main(args: Array[String]): Unit = {
      println(Ship("Endevour", 3, 4).distanceFrom(Ship("Falcon Millenium", 0, 0)))
   }
}
```

With case classes, all constructor parameters automatically become members, hence no need for member initialization. Also, the `new` keyword is no longer needed for creating new instances. Although very helpful, this is only one aspect of case classes. More importantly, case classes automatically provide value-base `equals`, `hashCode` and `toString` implementations. Additionally, they are sealed. In other words, case classes are perfect for creating immutable data types. Let's now compare the C# and Scala implementations of a class representing a two dimensional point so that you can see for yourself how nicer it is to write in Scala.

```scala
case class Point2d(x: Double, y: Double) {
    def move(dx: Double, dy: Double) = Point2d(x + dx, y + dy)
}

object Points {
   def main(args: Array[String]): Unit = {
      val a = Point2d(3, 4)
      val b = Point2d(5, 5)
      println(a.move(2, 1) == b)
   }
}
```

And now C#:

```csharp
using System.IO;
using System;

class Point2d {
    private double x;
    private double y;
    
    public Point2d(double x, double y) {
        this.x = x;
        this.y = y;
    }
    
    public Point2d move(double dx, double dy) {
        return new Point2d(x + dx, y + dy);
    }
    
    public override bool Equals(object obj) {
        if (obj == null)
        {
            return false;
        }

        Point2d p = obj as Point2d;
        if ((object)p == null)
        {
            return false;
        }

        return (x == p.x) && (y == p.y);
    }
}

class Program
{
    static void Main()
    {
        Console.WriteLine("Hello, World!");
    }
}
```

As you may now, [implementing equals is not trivial](https://msdn.microsoft.com/en-US/library/ms173147(v=vs.80).aspx). Not to mention that we would need to implement `GetHashCode`. In Scala we get the default implementation for free.
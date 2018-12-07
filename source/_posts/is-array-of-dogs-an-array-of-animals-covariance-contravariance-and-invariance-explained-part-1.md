---
title: >-
  Is array of Dogs an array of Animals? Covariance, contravariance and
  invariance explained - part 1
tags:
  - 'c#'
  - scala
url: 15.html
id: 15
categories:
  - Other topics
  - Scala
  - Tutorials
date: 2015-10-10 05:16:00
---

Welcome to the first post on my blog. I would like to dedicate it to a topic that sounds quite intimidating but is in fact quite simple to understand. There are already good explanations of type variance to be found on other blogs or [Stack Overflow](http://stackoverflow.com/) but I would like to take a broader approach and look at how different programming languages deal with it.

The problem
-----------

So, what is this cryptic title about? Let me start with this classic example in Java.

class Animal { }
class Dog extends Animal { }
class Cat extends Animal { }

MyList<Dog> dogs = new MyList<Dog>();
MyList<Animal> animals = dogs;

Would you expect this piece of code to compile? The answer depends on what operations are available on `MyList`. Let's assume that `MyList` is very similiar to `ArrayList` and it allows you to `get` and `add`.

class MyList<T> {
    public T get(int index) { /* get element at index */ }    
    public void add(T element) { /* add at the end */ }
}

Now, assuming that the questioned piece of code would compile, it would be perfectly valid to add a Cat to the list of Animals which is in fact a list of Dogs. This is not something we would want the compiler to allow.

animals.add(new Cat());
Dog dog = dogs.get(0); // we are expecting a Dog but we've got a Cat!

In this case, `MyList<Dog>` is **not** (does not inherit from) `MyList<Animal>`. We call `MyList` **invariant**. This is the kind of behaviour that we get in Java. Let's now assume that `MyList` is read-only and does not have an `add` method.

class MyList<T> {
    public MyList(List<T> original) { /* copy from original */ }
    public T get(int index) { /* get element at index */ }   
}

Now, the previous issue is no longer the case. If we call `animals.get()` we can get either a `Dog` or a `Cat` an we are ok with this. In such case, it makes sense to allow the questioned piece to compile. Hence, `MyList<Dog>` **is** (does inherit from) `MyList<Animal>` and we call `MyList` **covariant**.

Java
----

As stated before, in Java the below piece would not compile. In other words, generic types in Java are **invariant**. This is quite limiting when compared to other languages which allow you to specify variance for generics.

MyList<Dog> dogs = new MyList<Dog>();
MyList<Animal> animals = dogs;

Compiler output:

HelloWorld.java:22: error: incompatible types

However, there is an interesting exception to generic's invariance in Java. The below code will compile:

Dog\[\] dogs = new Dog\[\];
Animal\[\] animals = dogs;<br>

So, what happens when we try do add a Cat to an array of Dogs? Java gives us an exception (of course this will happen on runtime and not on compile time). So, arrays are **covariant** in Java! This is not a very elegant situation and the reasons behind it are mainly historic. There is a good explanation of this on [Stack Overflow](http://stackoverflow.com/questions/18666710/why-are-arrays-covariant-but-generics-are-invariant).

C#
--

Similarly to Java, C# would not allow us to compile below code:

class MyList<T> { }

MyList<Dog> dogs = new MyList<Dog>();
MyList<Animal> animals = dogs;

Compiler output:

error CS0029: Cannot implicitly convert type \`MyList' to \`MyList'

However, C# goes a step further and allows us to create variant generic interfaces. It is possible to mark a type parameter with the `in` keyword to make the generic interface covariant.

interface IMyList<out T> { }
class MyList<T> : IMyList<T> { }

IMyList<Dog> dogs = new MyList<Dog>();
IMyList<Animal> animals = dogs;

There are some nice examples of **contravariance** in C#. Since **covariance** means that you can use a more derived type than specified in type parameter, in **contravariance** you can use a more **generic** type than specified. It may seem a bit counterintuitive but let's look at the `Action<T>` type which represents a function that takes a parameter of type T and does not return anything.

Action<Base> b = (target) => { /* do something with target */ };
Action<Derived> d = b;
d(new Derived());

In this case, it makes sense to say that `Action<Base>` is `Action<Derived>`. `Action<Derived>` requires a prameter of type `Derived` so giving it an instance of something more generic (`Base`) is ok. [In the next post I will look at how variance is exploited in inheritance.](http://wordpress1653421.home.pl/home/platne/serwer16812/public_html/codewithstyle/?p=14)
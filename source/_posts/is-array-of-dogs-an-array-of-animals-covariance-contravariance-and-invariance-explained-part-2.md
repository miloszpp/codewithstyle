---
title: >-
  Is array of Dogs an array of Animals? Covariance, contravariance and
  invariance explained - part 2
tags:
  - csharp
  - java
  - covariance
url: 14.html
id: 14
categories:
  - Other topics
  - Scala
  - Tutorials
date: 2015-10-15 18:15:00
---

This post is a continuation of [Is array of Dogs an array of Animals? Covariance, contravariance and invariance explained - part 1](/is-array-of-dogs-an-array-of-animals-covariance-contravariance-and-invariance-explained-part-1).

Method overriding
-----------------

Type variance is not just relevant to generics but also to ineritance of regular, not generic, classes. When overriding a method in a class you usually make sure that it has the same argument types and return type. Note that it is not always necessary. For example, it makes sense for the overriding method to return a subtype of the return type of the original method.

```csharp
class AnimalList {
    Animal getAnimal() { return null; }
}

class DogList extends AnimalList {
    Dog getAnimal() { return null; }
}
```

The caller of `getAnimal` will expect an instance of `Animal`. Returning something more derived (a `Dog`) will be perfectly type safe. Therefore, we can say that return type of overriden method is **covariant**. Let's now look at argument types.

```csharp
class Animal {}
class Dog extends Animal {}

class DogComparator {
    bool isLarger(Dog x, Dog y);
}

class AdvancedDogComparator extends Comparator {
    boor isLarger(Dog x, Animal y);
}
```

`AdvancedDogComparator` is a specialized version od `DogComparator`. Just as `DogComparator`, it can compare two `Dogs` but it can do more than that. So, `AdvancedDogComparator.isLarger` must take at least a `Dog`, but it can also take the supertype of `Dog` \- an `Animal`. We can say that parameter types of the overriden method are **contravariant**. You may see an analogy here to how we deduced in the first post that it should be possible to make `MyList<T>` covariant as long as it does not have the `add` method. Return type covariance is supported both Java and C#. Argument type contravariance is not supported neither in Java nor C#. One more interesting case - if you create a covariant generic interface of type `T`, C# will not allow you to create a method that takes `T` in it.

```csharp
interface IMyList<out T> {
    void add(T el);
}

class MyList<t> : IMyList<t> {
    void add(T el) {
        Console.write(el);
    }
}
```

Compiler output:

```
error CS1961: The covariant type parameter \`T' must be contravariantly valid on \`IMyList.add(T)'
```

This is actually related to the contravariance of argument types when overriding methods. Any subtype of `IMyList` would have to override `add`. Therefore, the `T` would have to be contravariant but it is declared as covariant (the `out`) keyword which makes a contradiction.
---
title: 'Method overload resolution in C# 6.0: an interesting bug story'
tags:
  - csharp
  - csharp6
url: 179.html
id: 179
categories:
  - .NET
  - Articles
  - Other topics
date: 2017-04-07 13:40:58
---

Recently at work I've been looking into migrating our projects from VS2013 to VS2017. As part of the process we decided to move from C# 5.0 to C# 7.0. It turned out that after the switch some of our projects won't build anymore. I spent some time investigating the issue and found the outcome interesting so let me share my story with you.

### Problem

Below is the code that caused issues. It is an interface declaration with two overloads of a single Get  method.

```csharp
public interface IRepository<T> where T : class
{
	T Get(object id, params Expression<Func<T, object>>\[\] includeExprs);
	T Get(object id, params string\[\] includeExprs);
}
```

The code itself was fine. However, if we try to call it like this:

```csharp
repository.Get("some id");
```

strange things will happen. Under VS2013 the code will compile without issues. However, under VS2017 it will cause a compile error:

The call is ambiguous between the following methods or properties: 'IRepository<T>.Get(object, params Expression<Func<T, object>>\[\])' and 'IRepository<T>.Get(object, params string\[\])'

### Solution

Hmm, this totally makes sense. How is the compiler supposed to know which overload I mean? The solution is pretty simple - either don't use method overloading here or provide a third overload that takes no parameter list.

```csharp
T Get(object id);
```

I started to wonder which language feature introduced in C# 6.0 or C# 7.0 is responsible for this change of behaviour. After spending some time on fruitless thinking, I decided to ask a [question](http://stackoverflow.com/questions/42951282/breaking-change-in-method-overload-resolution-in-c-sharp-6-explanation) on StackOverflow. [Lasse](http://stackoverflow.com/users/267/lasse-v-karlsen) in his elaborate answer enlightened me that this is not strictly a change introduced by one of the new language features but rather a stricter behaviour introduced by the [Roslyn](https://roslyn.codeplex.com/) compiler which is shipped with Visual Studio starting from version 2015. I have later found this stated explicitly in [Roslyn documentation](https://github.com/dotnet/roslyn/blob/master/docs/compilers/CSharp/Overload%20Resolution.md#tie-breaking-rule-with-unused-param-array-parameters).

### Another problem

I decided to solve the issue by adding a third method overload taking only the id  parameter. In its implementation I picked one of the existing overloads randomly and called it with an empty parameter list:

```csharp
public T Get(object id)
{
	return this.Get(id, new string\[\] {});
}
```

How surprised I was to find out that some of our unit tests started to fail. After another couple of hours, I found the reason. It turned out that one of the overloads of the Get  method behaved differently with an empty parameter list (the first one would load all includes for an entity when given an empty list while the second one would load none).

### Conclusion

This was the moment when I realized how dangerous the before-Roslyn behaviour was. Given a call that was in fact ambiguous, the compiler would choose one of the overloads in a way that is by no means clear or intuitive. If by any luck it chose the same overload that you meant (as it happened in our case) you were relying on subtle, non-documented implementation detail of the compiler . The whole story thought me to be more careful when dealing with method overloading. The algorithm used for overload resolution is actually pretty complex and implements lots of rules (as exemplified [here](https://github.com/dotnet/roslyn/blob/master/docs/compilers/CSharp/Overload%20Resolution.md#tie-breaking-rule-with-unused-param-array-parameters) or in John's Skeet [C# in Depth](http://csharpindepth.com/) book). Always make sure that it's absolutely clear (both to you and the readers of your code) which method overload you mean.
---
title: A tour of C#
description: How I approached my first steps into the world of c#
date: 1-29-2023
categories:
  - dotnet
  - csharp
  - learning
image: C.NET.png
format:
    html:
        toc: true
        toc-location: left
        number-sections: true
---

# Motivation

I have primarily worked in python and typescript and I feel fairly comfortable solving most problems in those languages. There's also not many things you can't do with that combination so my main motivation for picking up C# was to expose myself to different patterns and coding practices.

My goal was to get a general overview and try to map known concepts to my exisiting knowledge from languages I've worked with, and then learn what concepts were new so I could uncover the “unknown unknowns”.

I also wanted to make sure I didn’t go too far down any rabbit holes, since I wanted to start writing C# code and build something as soon as possible. So there was a tradeoff between spending too long on theory vs jumping straight into building things without taking advantage of C# features. I didn’t want to just write python code with C# syntax.

# First steps
I started by going through the [C# for Beginners playlist](https://www.youtube.com/playlist?list=PLdo4fOcmZ0oVxKLQCHpiUWun7vlJJvUiN) on YouTube but it was targeted towards beginners without any programming experience so it felt too basic and slow paced, I decided to move on after video 9.

I decided to go to c# docs so I could go through things at my own pace, I found the [C# language tour page](https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/) covered most things I needed to get started.


# A tour of C#

The following sections are my notes and thoughts from going through the sections on the language tour page. There are likely to be misunderstandings here that i'll look back on and cringe at - but I find it useful to document my thought process at the time.

## Introduction

- Hello world has a lot more boilerplate than I'm used to!
- Some of the terminology around interfaces and types seems familiar from learning typescript
- `namespace` looks like a more explicit mechanism to how you would use the directory structure in python to import things

## Types

- Inheritance, generics are familiar concepts from python and typescript!
- interfaces seem similar to an abstract class?
    - seems to be a common enough question (interfaces are a contract, abstract classes can implement some of the methods for you)
    - [Interfaces vs. abstract classes on stack overflow](https://stackoverflow.com/questions/747517/interfaces-vs-abstract-classes)

## Program building blocks

This section took me a while to get through - loads of new concepts to learn!

- So many different types of members in a c# class!
- I previously thought of properties as the same as fields and constants
    - it's referred to as an “action” so it sounds like a getter and setter (see later notes for clarification here)
- Seems like indexers are a way to keep track of all instances of a class?
    - saves you from having to do this yourself, i.e. using a class variable and incrementing by one/appending to list in the constructor
- Events look interesting, reminds me of the observer design pattern
- Accessiblity of members is a concept i’m familiar with but in python it’s more of a convention rather than an inbuilt feature when you use `_variable_name` to show something should not be accessed outside the class
    - `internal` , `protected internal` and `private protected` seem a bit more advanced so i’ll let those go for now
- `static` just looks like how you’d use `@classmethod` in python
- The concept of passing by value vs reference is not new, but the `ref` keyword reminds me of using seeing something similar back when I had to use VBA (the `ByRef` keyword)
- I can’t see reasons to use `out` instead of just returning arguments, will be interesting to see use cases of that where it has advantages over a return value
    - [Stack overlow answer on which is better, return value or out parameter?](https://stackoverflow.com/a/810806)

- I had to read the section on `virtual` and `abstract` methods a few times, this made it click:

> An abstract method is a virtual method with no implementation.

Does this mean we cannot override methods in a derived class unless they have a virtual method in a base class?

- The answer is yes: [Versioning with the Override and New Keywords - C# Programming Guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/versioning-with-the-override-and-new-keywords?redirectedfrom=MSDN)

Method overloading feels a bit dodgy, the example in the docs has a “catch-all” in `static void F<T>(T x)`: 

```csharp
class OverloadingExample
{
    static void F() => Console.WriteLine("F()");
    static void F(object x) => Console.WriteLine("F(object)");
    static void F(int x) => Console.WriteLine("F(int)");
    static void F(double x) => Console.WriteLine("F(double)");
    static void F<T>(T x) => Console.WriteLine($"F<T>(T), T is {typeof(T)}");            
    static void F(double x, double y) => Console.WriteLine("F(double, double)");
    
    public static void UsageExample()
    {
        F();            // Invokes F()
        F(1);           // Invokes F(int)
        F(1.0);         // Invokes F(double)
        F("abc");       // Invokes F<T>(T), T is System.String
        F((double)1);   // Invokes F(double)
        F((object)1);   // Invokes F(object)
        F<int>(1);      // Invokes F<T>(T), T is System.Int32
        F(1, 1);        // Invokes F(double, double)
    }
}
```

Why does `F(”abc”)` not require you to declare the type parameter .e.g `F<string>("abc")` like we see in `F<int>(1)`?

- Static constructors are new concept, why would you want to initialise a class?
    - [Static Constructors - C# Programming Guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/static-constructors#usage): shows examples for initialising a log file, runtime checks

- properties have get and set accessors like: `public int Property {set => ... ; get=> ...;}` (can change read, write accessbility by including or leaving out `get` or `set` accessor)
- Indexers seem like the `__getitem__` method in python, i.e. it provides a way to index the class like `MyClass[SomeIndex]`
- Events fields store reference(s) to `delegate` - delegate just seems like a bunch of function references that can be passed around?
    - [Delegates - C# Programming Guide](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/delegates/)

How events are used:
- Declare events in the class and the dispatch the events:

```csharp
public class MyList<T>
{
	// ...
    // event dispatcher:
	protected virtual void OnChanged() =>
        Changed?.Invoke(this, EventArgs.Empty);

	public event EventHandler Changed;
}
```

Add event handlers:

```csharp
class EventExample
{
    static int s_changeCount;

    // event handler definition
    static void ListChanged(object sender, EventArgs e)  
    {
        s_changeCount++;
    }
    
    public static void Usage()
    {
        var names = new MyList<string>();
        names.Changed += new EventHandler(ListChanged); // adding the event handler
        names.Add("Liz"); // in the Add function we invoke onChanged()
        names.Add("Martha");
        names.Add("Beth");
        Console.WriteLine(s_changeCount); // "3"
    }
}
```

- Finalizers are for when objects are garbage collected - probably won’t need this anytime soon, `using` is recommended for object destruction.

## Major Language Areas

`IEnumerable<T>` interface implemented by collection types - used for iteration

- Static arrays allow you to define the size of an array `int[] t = new int[3];`
    - leaving out the `3` will make it dynamic (add as many items as you like)
- Delegate types: reference to method with parameter list and return. Lets you pass methods around
    - can reference static or instance methods (or lambda/anonymous functions)
        - as long as parameter and return types match
        - with instance methods, `this` is available and references the instance method’s object
- async/await syntax looks similar to typescript, will need to play around further to understand differences
- Attributes let you attach metadata to your code with square bracket notation:

```csharp
[Attribute]
class Example(){}
```

- These can then be accessed at runtime with reflection (used for [routing to controller actions in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/routing?view=aspnetcore-7.0#ar6)

# Next steps

Time to write some C# code! I want to get started with some basic coding exercises to get a feel for the syntax before I build a full application. The [C# 101 notebooks](https://techcommunity.microsoft.com/t5/educator-developer-blog/using-visual-studio-notebooks-for-learning-c/ba-p/3580015) seem like a good place to start with that.

Check out the next post to see how I setup .NET notebooks and why I find them useful to explore the language.

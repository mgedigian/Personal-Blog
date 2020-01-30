---
layout: post
title: Pure Function vs Referential Transparency
date: 2020-01-30 12:16
summary: Referential Transparency might not equal to Pure Function
categories: functional-programming tech programming scala
tags: functional-programming tech programming scala
---

<img src="{{site.baseurl}}/images/pure-function-vs-referential-transparency/Pure vs Referential Transparent.png" alt="Pure vs Referential Transparent">

I was teaching functional programming in scala courses to my colleagues today and presented some problems with pure function and referential transparency. One of the problems, which is rather tricky, tests if the current function is pure or not. It goes like this:

```scala 
val one = 1
def method(num:Int) : Int = {
  one + num
}

```
The problem gives a fascinating discussion of where the function is pure or not. One of the colleagues said that it is a pure function because, with the same input, you always get the same output. However, in pure function, it mentioned in the definition that it has no "back doors," which means it doesn't depend upon any hidden value outside of the function scope. 

Is this function pure or not?

I gave an answer mentioning that based on the pure function definition, it is not pure.

I was wrong.

The answer to that is that the function is pure, and it is referential transparency.

Before we go into pure function and referential transparency, let's talk about what those terms are.

## What is Pure Function
A pure function depends on the input parameters and its algorithm to produce its result.

For a function to be pure, the output depends on the input function, and that you don't have to worry that the same input function produces a different output function.

The pure function also functions that have no side effects. What are side effects? 

You can think of side effects as anything that makes the function unpredictable. You want your function to be predictable so that you never need to scratch your head with the output of the function.

Usually, a pure function is a function that 100% produces the same output in run time.

Foreach is only use for its side-effect.
```scala 
def foreach(f:(A) => Unit):Unit
```

The fact that foreach produces nothing in return is not a pure function, because the function is meant to produce side effects, such as `println` or modify certain variables inside.

Side effect meaning it relies on any external I/O. A pure function cannot rely on input from files, databases, web services, UI; it cannot produce an output such as writing to a file, database, or web services.

One other important thing about Pure function is that if a function is pure, it is also Referential Transparent.


## What is Referential Transparency

If you never touch Functional programming before, chances are you never heard of this term. I never heard of Referential Transparency until I work on projects in Scala. OOP has been very popular in software development, and the 4 principles are widely discussed in software engineer interviews to test a candidate's understanding of OOP. However, more and more application is built upon a functional paradigm, and Referential Transparency is like one of the principles of Functional Programming.

Referential transparency means you can replace the function with its value and get the same output. 

You can construct a map with the function as the key and the value as another representative of the key, and you can be sure that the key-value pair always persist no matter what scenario.

Take this example:
```scala
def square(x:Int) : Int = x * x
```

When you execute `square(2)` you will get 4. You can replace `square(x)` with `x*x` in the run-time, and it will get you the same output without really changing the behavior of the program. Therefore, `square(1) + square(2)` can be replace with `1*1 + 2*2` and it will still yield the same value.

An example of non-referential transparent:
```scala
def addRandom(x:Int) : Int = {
  (new util.Random).nextInt(10) + x
}
```

The example code above is not referentially transparent, because you cannot replace `addRandom` with any defined value inside that function to get the same output - meaning `addRandom(1) ≠ addRandom(1)`.

Another example of non-referential transparent is Future:
```scala
for {
  _ <- Future{println("Future1")}
  _ <- Future{println("Future1")}
} yield {}
```

Future statements are <a href="http://scalapro.net/scala-futures-traverse-and-side-effects/" target="_blank">eager</a> in nature. Therefore, it prints twice to the console. However, if Future is referentially transparent, it also prints twice if you do this:
```scala
val f = Future{println("Future1")}
for {
  _ <- f
  _ <- f
} yield {}
```
However, in this case, there Future only print once. It indicates that the Future is not referentially transparent.


## Is Pure Function equal to Referential Transparency?

A pure function is a subset of Referential Transparency. Why? Because to be pure, you need not have side effects and referential transparent. However, the other way around might not be right. 

Take this function:
```scala 
def method(x:Int) : Int = {
  println(x)
  x
}
```

<img src="{{site.baseurl}}/images/pure-function-vs-referential-transparency/Referential Transparency.png" alt="Referential Transparency.png">

This function is referentially transparent because you can change the underlying method with a print statement and x; and it yields the same result. `method(1)` will always equal to `method(1)`, and `method(42)` can be replaced as `print(42); 42`. However, this function is not pure because it has an IO, print statement.

## Main Takeaway
- If a function is not referential transparency, be aware of some unexpected output from a function with the same input is being called multiple times. 
- A pure function is more constraint than referential transparency.

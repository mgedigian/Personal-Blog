---
layout: post
title: What is the Difference Between Function Literal and Values in Scala
date: 2020-01-14 08:40
summary: They are similar yet not the same. Therefore, becareful with some bugs you encounter when using them.
categories: programming tech scala software-development
tags: programming tech software-development scala
---

<img src="{{site.baseurl}}/images/what-is-the-difference-between-function-literal-and-values-in-scala/Function Literal vs Function Values.png" alt="Function Literal vs Function Values">

I encountered some bug a couple of weeks ago when trying to play around with partially applied function.

Partial Applied Function is a function that partially created, leaving some of the arguments not applied. While you invoke the function, you can state some of the arguments, and the left argument is supplied when it is required.


Take this for an example:
```
def add(a: Int, b: Int): Int = a + b 

val addition = add _

addition(1,2) // 3
```

However, when I try to do it in function literal:
```
val add = (a:Int,b:Int) => a+b
// add: (Int, Int) => Int = $$Lambda$844/1630986748@1736c1e4

val addition = add _
// addition: () => (Int, Int) => Int = $$Lambda$845/1518517336@4aab7195

addition(1,2) // this doesn't work

addition()(1,2) // 3
```

Makes we wonder the difference between function literal and function value.

In the scala book, the definition of function literal and function value is at the compile and run time. 

For function literal, (anonymous function or val function), will be compiled into Function2 trait under the hood. Usually, they snap it with an apply method that also is run as a def function (value function).

Therefore, if you assign to a val as an anonymous function, the scala will transform it into a singleton object that has a function apply.

```
final class anonFun extends scala.runtime.AbstractFunction2 {
    final def apply(a: Int, b:Int) = a + b
}

val add = new anonFun();
```

When you assign the function literal to another variable as `a`, the variable will also be created with another object that has another `apply` method - causing it becoming a higher-order function `() => () => something`.

However, function values, def function, in scala is served as a regular method in Java. Therefore, when you write `def` and assigned it to another variable, that variable is getting the method that has access to other members variable in that class. Therefore, the compiler does not create a `Function2` trait and an apply method.

It creates an instance method of the object 
```
package <empty> {
  object Main extends Object {
    def add(a: Int, b:Int): Int = a.+(b);
    def <init>(): Main.type = {
      Main.super.<init>();
      ()
    }
  }
}
```


## Main Takeaway:
- Function literal, val function, is compiled into a class when instantiated at run-time is a function value. _It becomes the object itself_. 
- If you assign another variable as a partially applied function to a function literal, it created another object literals that have an apply method slap into it. Therefore, creating a higher-order function.
- If you use function value, a def function compiles into a method in Java. Therefore, it becomes an `instance method` of an object. 


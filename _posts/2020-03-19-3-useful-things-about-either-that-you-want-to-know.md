---
layout: post
title: 3 Useful Things About Either That You Want To Know
date: 2020-03-19 12:22
summary: Either helps you construct better error handling 
categories: monad programming functional-programming scala
tags: monad programming functional-programming scala
---

![Photo by Jon Tyson](https://images.unsplash.com/photo-1508591360875-10163ed98c8e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=750&q=80)

Either is a new type of class that I get to learn when I was learning Scala. As I learn more in-depth and more profound about functional programming, Either is not just a container that stores two values of left and right, but it is a Monad.

One of the reasons to use Either type constructor is not to create any surprising output when running the program. Without Either, your function is not predictable. You may encounter some unexpected side effects when an exception is thrown. For instance,  instead of blows up the exception unexpectedly, Either can return a Left (failure case) or Right (success case). Therefore, the function is predictable to the caller, letting the caller knows what potential result may happen. 

In this article, I want to share some useful Either characteristic that I think is beneficial to know about Functional programming.

## Either is a Monad

In Scala 2.11 and earlier, many people didn't consider Either a Monad because it didn't have a map and flatMap methods.

This makes it very hard to compose sequence using for comprehension.

```scala
val either1: Either[Exception, Int] = Right(1)
val either2: Either[Exception, Int] = Right(2)

for {
  one <- either1.right
  two <- either2.right
} yield one + two


```

However, in Scala 2.2, Either was redesigned to have Functory like features.

The modern Either decides that the right side is the success case. Thus, it supports the map and flatMap.

It makes the for-comprehension much more pleasant:

```scala
val either1 : Either[Exception,Int] = Right(1)
val either2: Either[Exception, Int] = Right(2)

for{
  one <- either1
  two <- either2
} yield one + two
```

It transformed Either from unbiased type to a right-biased.


## Either is right bias

To be a Monad type, you need to be able to apply a map or flatMap to the value. However, Either is categorized as the Sum Type in <a href="https://edward-huang.com/functional-programming/2019/12/30/what-is-an-adt-algebraic-data-types/" target="_blank">ADT</a>. That means it has two possible values, a right, and a left. Another similar Monad type to Either includes Future (Success, Failure), Option (Some, None).  These Monads hold two values, usually a happy and sad scenario.

Which value should the function apply?

This is where the right bias came into the picture. Right biased means that functions such as `map` and `flatMap` only apply to the "right side" or the "happy scenario", leaving the other side untouched.

Up until Scala 2.12, Either was unbiased, which means the `map` and `flatMap` function don't know which value to apply. You have to use "right projectable" to make it right-biased and _flatmappable_. 

Either being un-biased is suitable for validation functionality. However, with right-biased, Either can be very handy in creating a sequence of operation.

For instance, take an example of validating an object.

```scala
case class CarEngine(sound: String)
```
You want to validate this engine sound to see if it is a car. It can be done in Either by throwing an error if it is not a sound of `vroom`. Then, you want to return the instance of that object if it is a valid car engine.

```scala
def validCarEngineSound(car:CarEngine): Either[Exception, CarEngine] = {
  car.sound.toLowerCase match {
    case "vroom" => Right(car)
    case _ => Left(new Exception("not a valid car sound"))
  }
}
```

Now, the beauty of right-biased is doing some other operation after the validation, deferring the error handling. 

Let's say you want to wire the sound of the `CarEngine` and add it to Toyota class because Toyota needs to restrict its car engine sound to be a certain way.

```scala
// TOYOTA class define somewhere in the class
val engine = CarEngine("vroom")
val result = validCarEngineSound(engine).map{ case CarEngine(sound) => wireToToyota(sound) }

result match {
  case Left(err) => println(err.getMessage)
  case Right(_) => println("The engine is correct and is wired to Toyota")
}
```

If the value is not valid, then it defers it and returns in the result. However, if it is successful, then it executes the value inside the `map` function.

## Fold

The last one is the operation of fold a list on an Either can be tricky. 

Take this example:
```scala
List(1,2,3).foldLeft(Right(0)) { (accumulator, num) => 
  if(num > 0) {
    accumulator.map(_ +1)
  } else {
    Left("Negative. Stopping!")
  }
}
```

Since Left and Right is a subtype of Either, it throws an error saying that the type is a mismatch. Therefore, you need to specify the type parameters for `Right.apply` so the compiler can infer the left parameter as `Nothing`.

It would be best if you cast them. Cats provided a smart constructor that wraps the Left and Right value as Either with `asRight`.

```scala
implicit class EitherOps[A](v:A) {
  def asLeft[B]: Either[A,B] = Left(v)
  def asRight[B]: Either[B,A] = Right(v)
}

List(1,2,3).foldLeft(0.asRight[String]) { (accumulator, num) => 
  if(num > 0) {
    accumulator.map(_ +1)
  } else {
    Left("Negative. Stopping!")
  }
}

```

Another one of the useful foldLeft on an Either provide a left and a right scenario.

This is taken from the scala library function definition.
```scala
type Either[+A,+B]
def fold[C](fa:A => C, fb:B => C):C
```

It applies a `fa` when the value is a right, and `fb` when the value is a b. It is convenient when you want to do a left and right value without doing a pattern-matching.

## Takeaway
- Either is a monad, which has a map and flatMap functionality. We don't notice it now how handy Either becomes after it becomes a Monad.
- Either is right-biased, meaning the `map` and `flatMap` method can execute if the value is a "right" or "happy scenario". It leaves the "left" scenario untouched.
- The fold is another way to write function execution on Either pattern matching.


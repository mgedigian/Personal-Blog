---
layout: post
title: Let Me Teach You How To Implement Monad With Cats
date: 2020-03-16 22:18
summary: Create your own Custom Monad with Cats Library
categories: functional-programming scala programming monad
tags: monad functional-programming scala tech monad cats
---

![Photo by alireza mahmoudi](https://images.unsplash.com/photo-1573027167082-4a567857689b?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=668&q=80)

Monad is one of the most common abstractions in Scala. We often used them in Scala and any other languages, but we often didn't know them by name.

When I was learning functional programming, besides understanding the rules of referential transparency and pure, I was utterly confused with what Monad, Functor, Semigroup, Monoid are. 

In this article, I want to share about how you can create your Monad with the Cats library by implementing `pure`, `flatMap`, and `tailRecM`. 

Before that, I want to share a little bit about map and flatMap, and a brief definition of Monad, before we dive deep into implementing a Monad for a custom type.

## What is a Monad
Simply put, Monad is a mechanism to sequence computations. It is anything that has a flatMap functionality.

Monad is a subset of Functor, which has the map functionality. Therefore, all Monad is also Functor, and they can apply a map on the mechanism.

Some examples of Monad included - List, Options, Future, Either. It is often used that Scala has a special syntax to support monad operation - for comprehension.

Let's jump right into the difference between map and flatMap. Then, implement a custom type to a Monad.

## Map vs FlatMap
When you look at map and flatMap from the perspective of javaScript, it is a function that iterates over an iterator transforming those elements inside the iterator and return a completely new array with the transformed iterator.

For instance, you can map a list of integers and evoke the call-back function of transforming the element inside a string. 

```scala
val lst = List(1,2,3)
val stringList = lst.map{el => s"${el}" }
println(stringArr);
```

The output of the `stringArr` will be `List("1","2","3)`. What did map do over there?

The map function fundamentally transforms each integer element in the List to a string. Then, it returned a new list and assigned it to `stringList`.

In Scala, the map holds a different meaning than just iterating an iterator. The map is not an iteration pattern. Put, flatMap, and map is a way to transform a sequence of computations. The map and flatMap execute a sequence of computation on the values by ignoring the complications that are dictated by the relevant data types. 

You cannot only do a map in a List but also an Option, Future, and Either. 

What map does to these data types is that it peel out the outer layer of the data types, List, and apply the call-back function into each of the elements in that List. Then, once it is finished with the operation, it covers that value inside with the existing data types that it peels (List).

The same goes for Option. Map applies its function to the value inside of the Option. The resulted value is still an option, but the value inside the Option is changed, transformed.

<img src="{{site.baseurl}}/images/let-me-teach-you-how-to-implement-monad-with-cats/Map vs FlatMap.png" alt="Map vs FlatMap">


Map is restricted in a way that they only allow the complication to occur at the beginning of the computations. Flatmap goes even further, not only that you can only transform the value inside of the data types, but also chain it into a sequence of computations.

When you evoke a flatMap, it mainly does the same with `map`, but then it calls `flatten` to flatten out the resulted value. One example is when you try to flatten a 2D List to a 1D list. 

```scala
val twoDList = List(List(1,2), List(3,4), List(5,6))

twoDList.flatMap(el => el)
```

The above code is the same as doing `twoDList.map(el => el).flatten`.

Let's take another one with Option.
```scala
def divide(a:Int, b:Int): Option[Int] = if(b == 0) None else Some(a/b)

Some(1).flatMap{ one =>
  Some(2).flatMap{two =>
    divide(one,two)
  }
}
```

In this example, flatMap takes off the intermediate complication. The flatMap of Option takes care of the intermediate Options into account. The function that is passed inside flatMap specifies the application-specific of the computation. The flatMap function in the above example short-circuits the operation if any intermediate value is a None.





## How do you define a Custom Monad
In Cats library, you can define a custom monad by merely implementing these 3 methods:
- flatMap
- pure (Applicative)
- tailRecM

We have talked about flatMap. Pure is a function that is provided by Applicative. Applicative also extends Functor, which gives Monad a map method. tailRecM is an optimization used in Cats library to limit the amount of stack space used. 

When you can implement tailRecM tail recursive, Cats library can guarantee stack safety in large operations such as folding an extensive list. However, if you cannot make the tailRecM tail recursive, cats cannot be guaranteed if it is stack safe in extreme use cases. 

Let's make a CustomMonad class a Monad.

```scala
case class CustomMonad[A](value:A)
```

Before, you need to import Cats library in the build.sbt in order to impelement custom monad. 
```scala
// build.sbt
lazy val customMonad = project.in(file("customMonad"))
  .settings(
    name := "Custom Monad",
    commonSettings,
    libraryDependencies ++= Seq(
      "org.typelevel" %% "cats-core" % "2.0.0"
    )
  )
```

First, we implement the pure, which transforms a value to an option.

```scala
override def pure[A](x: A): CustomMonad[A] = CustomMonad(x)
```

Here is the flatMap function:
```scala
override def flatMap[A, B](fa: CustomMonad[A])(f: A => CustomMonad[B]): CustomMonad[B] = f.apply(fa.value)
```
Since flatMap function `f` takes in an `A => CustomMonad[B]`, we just need to apply the function to `fa`.


Lastly, let's implement the tailRecM function.
```scala
 @tailrec
override def tailRecM[A, B](a: A)(f: A => CustomMonad[Either[A, B]]): CustomMonad[B] =        f(a) match {
      case CustomMonad(either) => either match {
        case Left(a) => tailRecM(a)(f)
        case Right(b) => CustomMonad(b)
      }
    }
```
The tailRecM function will need to recursively called itself, until the result of `f` returns a `Right`. Therefore, the Left function will call the `tailRecM` again, because it is not the end of the sequence.

Combine the above implementation all together:
```scala
import cats.Monad

 implicit val customMonad = new Monad[CustomMonad] {
    override def pure[A](x: A): CustomMonad[A] = CustomMonad(x)

    override def flatMap[A, B](fa: CustomMonad[A])(f: A => CustomMonad[B]): CustomMonad[B] = f.apply(fa.value)

    @tailrec
    override def tailRecM[A, B](a: A)(f: A => CustomMonad[Either[A, B]]): CustomMonad[B] = f(a) match {
      case CustomMonad(either) => either match {
        case Left(a) => tailRecM(a)(f)
        case Right(b) => CustomMonad(b)
      }
    }
  }
```

Once you finish implementing this class, don't forget to import `cats.implicits._` to retrieve the implicit in your `main` function.

You can execute `CustomMonad` with Functor like syntax:
```scala
import cats.implicits._


object Main extends App {

  val endResult = for {
    a <- CustomMonad(1)
    b <- CustomMonad(2)
  } yield {
    a + b
  }
  println(endResult)
}

```

Once you finished with implementing your custom monad, you can import cats <a href="https://typelevel.org/cats/typeclasses/lawtesting.html" target="_blank">law dependency</a> to check if your custom monad abides by the <a href="https://wiki.haskell.org/Monad_laws" target="_blank">Monad Law</a>. This <a href="https://stackoverflow.com/questions/39561525/how-to-test-monad-instance-using-discipline" target="_blank">StackOverflow questions</a> shows how to test Monad's Law with Discipline.


## Takeaway
- Monad is a mechanism to sequence operations and anything that can be flatMap.
- Map and flatMap are a way for Monad to sequence the operation without having to care for any complication of the data types and intermediate operation.
- Implement a custom Monad by defining the flatMap, pure, and tailRecM function in Cats library.

The source code is on <a href="https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/customMonad" target="_blank">Github</a>.

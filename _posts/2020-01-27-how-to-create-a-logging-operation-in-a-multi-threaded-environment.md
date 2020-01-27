---
layout: post
title: How to Create a Logging Operation in a Multi-Threaded Environment
date: 2020-01-27 09:49
summary: How to create a Monad that helps your logs to not resulted in interleaved messaged from different context.
categories: scala cats programming functional-programming
tags: scala cats programming functional-programming software-development tech
---


![Photo by leanncaptures](https://images.unsplash.com/photo-1559755693-edc7d9f57b5b?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2550&q=80)


Imagine that you are writing function of calculating the Fibonacci series like this and add a print statement for debugging purposes:

```scala

def fib(n:Int) : Int = {
  if(n == 0 || n ==1) {
    println(s"base case : $n")
    n
  }
  else {
    println(s"add fib(n-1) + fib(n-2) $n")
    fib(n-1) + fib(n-2)
  }
}

```
Then, you run the following function:

```scala
fib(5)

// interpreter
add fib(n-1) + fib(n-2) 5
add fib(n-1) + fib(n-2) 4
add fib(n-1) + fib(n-2) 3
add fib(n-1) + fib(n-2) 2
base case : 1
base case : 0
base case : 1
add fib(n-1) + fib(n-2) 2
base case : 1
base case : 0
add fib(n-1) + fib(n-2) 3
add fib(n-1) + fib(n-2) 2
base case : 1
base case : 0
base case : 1
fib: (n: Int)Int
res0: Int = 5
```

A condition of executing multiple functions synchronously, it works just fine:

```scala
scala> :paste
// Entering paste mode (ctrl-D to finish)

fib(5)
fib(4)
fib(3)

// Exiting paste mode, now interpreting.

add fib(n-1) + fib(n-2) 5
add fib(n-1) + fib(n-2) 4
add fib(n-1) + fib(n-2) 3
add fib(n-1) + fib(n-2) 2
base case : 1
base case : 0
base case : 1
add fib(n-1) + fib(n-2) 2
base case : 1
base case : 0
add fib(n-1) + fib(n-2) 3
add fib(n-1) + fib(n-2) 2
base case : 1
base case : 0
base case : 1
add fib(n-1) + fib(n-2) 4
add fib(n-1) + fib(n-2) 3
add fib(n-1) + fib(n-2) 2
base case : 1
base case : 0
base case : 1
add fib(n-1) + fib(n-2) 2
base case : 1
base case : 0
add fib(n-1) + fib(n-2) 3
add fib(n-1) + fib(n-2) 2
base case : 1
base case : 0
base case : 1
res2: Int = 2
```
However, if the function is wrapped in an asynchronous manner, it will be really hard to tell which log is associated to which. In this scenario, how can we debug an operation that is in asynchronous manner?

## Writer Data Types to the Rescue
Cats have a writer data type class that can help you rescue by attaching your log statement with the underlying result value so that you can understand log statements asynchronously.

In Cats, Writer data types definition is: `Writer[L, V]`.

L is the logging type collection <a href="https://typelevel.org/cats/typeclasses/monoid.html" target="__blank">Monoid</a> that you want to have; in this case, we can set L as a `Vector[String]`. 

V is the result type of the function operation - in this case, we can set V as `Int` since we are returning an integer type from the Fibonacci.

### Initialization
Once you create know what is `L` and `V` in the definition, you can create your Writer Data Type like this:
```scala
import cats.data.Writer
import cats.implicits._

Writer(Vector("log1", "log2"), 0)
```

or 

```scala
import cats.data.Writer
import cats.implicits._

0.writer(Vector("log1", "log2"))
```

If you saw the repl, or the result, you will realize that it is not `Writer[L, V]`, but return a `WrtierT[Id, L, V]`. It is because cats use type alias to derive the value of `Writer` from `WriterT`. In this post, we are going to talk about how to use Writer. Therefore, you can ignore the details and treat the type `WriterT[Id, L, V]` as `Writer[L, V]`.


### Having log value but no result
If there is a log but no result we can use `tell`:
```scala
Vector("msg1", "msg2").tell()
```

We can also extract both the output and the logs at the same time with `run`:

```scala
val writer = Writer(Vector("something"), 0)
val (log, result) = writer.run
```

### Extract Result value and Log type
Extract the result and log with `value` and `written` respectively:
```scala
val a = Writer(Vector("msg1"),0)
val log = a.written
val result = a.value

println(s"log: $log result: $result")
// log: Vector(msg1) result: 0
```


## Composing and Transforming Writers
Since Writer is a <a href="https://typelevel.org/cats/typeclasses/monad.html" target="__blank">Monad</a>, you can do operation on Writer with `map` and `flatmap`.

`flatMap` combines the log type and also the result type together from the source Writer and the result of the sequencing function.

Therefore, it is a good practice to put a Log type that has an efficient of append and concatenate method, such as Vector:
```scala 
val res = for {
    a <- Writer(Vector("a"), 1)
    _ <- Vector("c").tell
    b <- 3.writer(Vector("3", "b"))
  } yield {
    println(s"a $a") // 1
    println(s"b $b") // 3
    a + b // 4
  }

println(res) //WriterT((Vector(a, c, 3, b),4))
```

Note that the `tell` method will preserve the original Writer and append the "c" to the source Writer, which is "a".

The result of the Writer is based on what will be computed after the `yield` function. If there is no addition after `yield`, and only `a`, for instance, the end result will be `WriterT((Vector(a,c,3,b),1))`.

### Transforming Writer
We can change the Log type to all upperCase by using `mapWritten`:
```scala
// .. take example from previous res example
val upperCaseLog = res.mapWritten(previousLog => previousLog.map(_.toUpperCase))
  println(upperCaseLog) 
  // WriterT((Vector(A, C, 3, B),4))
```

You can also transform both type by using `mapBoth`:
```scala 
val newWriterValueAndLog = res.mapBoth{ (log,res) =>
    (log :+ "appending z", res+12)
  }
println(newWriterValueAndLog)

WriterT((Vector(a, c, 3, b, appending z),16))

```
Swap the log type and the result using `swap`:
```scala 
val swappedWriter = res.swap
println(swappedWriter)
// WriterT((4,Vector(a, c, 3, b)))
```

Last but not least, reset the log value in Writer using `reset`:
```scala
val resetWriter = res.reset
println(resetWriter)

// WriterT((Vector(),4))
```

### Writer in Action
Now you have read through this far and know what a Writer is, let's refactor our code to incorporate Writer in it:

First, let's add a timeOut function to set up the asynchronous environment.
```scala
 def timeout[A](body: => A):A = try {
    body
  } finally Thread.sleep(100)
```
Then we set a Type alias of LogFib from Writer:
```scala 
type LogFib[A] = Writer[Vector[String], A]
```

We change the Fib function to return a `LogFib[Int]`:
```scala
def fib(n:Int): LogFib[Int] = {
    timeout(
      if(n == 0 || n ==1) {
        n.writer(Vector(s"base case : $n"))
      }
      else {
        for {
          _ <- Vector(s"add fib(n-1) + fib(n-2) $n").tell
          fib1 <- fib(n-1)
          fib2 <- fib(n-2)
        } yield fib1 + fib2
      }
    )
  }
```

Then you can run it like this :
```scala 
import scala.concurrent.duration._
  implicit val ec :ExecutionContext = scala.concurrent.ExecutionContext.Implicits.global
  val fibRes = Await.result(Future.sequence(Vector(
    Future(fib(5)),
    Future(fib(4)),
    Future(fib(3))
  ))
  , Duration.Inf)


  fibRes.toList.map(w => {
    val (logging, endResult) = w.run
    println(s"logging $logging endResult $endResult")
  })
```

## Takeaway
- Writer Data Type is useful for a logging operation in a multi-threaded environment
- The Writer Log is tied to the result. Therefore, it is an excellent way to record the sequence of multi-threaded computation.


All the example information are in <a href="https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/catsWriterType" target="__blank">github</a>


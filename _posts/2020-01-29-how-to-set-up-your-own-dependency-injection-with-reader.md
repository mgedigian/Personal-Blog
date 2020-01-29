---
layout: post
title: How to Set up your Own Dependency Injection with Reader
date: 2020-01-29 09:48
summary: Using Reader Monad you decouple your validation system
categories: functional-programming scala cats monad tech
tags: functional-programming scala cats monad tech
---

![Photo by Frank Wang](https://images.unsplash.com/photo-1537151242758-331155dcf21b?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80)

There are many ways to create dependency injections. Dependency injection helps decoupled one object from another, bypassing a dependency to a framework or an object during run time. Therefore, the client doesn't need to find what dependency needs to supply to the object or framework - instead, the system tells the client what dependency they need to supply to the object.

In Scala, there are many ways to create dependency injection. Today, I will talk about using Reader as a Monad in Cats for creating dependency inject, and how Reader can help you quickly create configuration validation for your system.


Imagine if you need to create a validation system that can easily pick and choose various kinds of operations for validating configuration in a system. You can create a method to validate each component in the configuration.

```scala 
def isCorrectEmail(id:Long, email:String, repo: Repository) : Boolean = {
  if(repo.findId(id)) repo.getId(id) ....
}
```

In this case, if one day a new configuration attribute is needed to validate, or a specialized check from the configuration to make it pass, you will need to go to the code and change the methods. 

With more changes in the configuration file, you need to go to the file and change the code each time. Moreover, every single check is coupled with one another, making it hard to update in the future.

## Reader to the rescue
Cats Reader Data type can help you chain multiple operations together, and produce a significant computation that accepts a configuration as one parameter. It would be best if you inputted the configuration, and it runs as we specified.

Let's take a look at the definition of `Reader[A, B]`:
```scala 
object Reader {
  def apply[A,B](f:A => B) : Reader[A,B] =  ReaderT[Id,A,B]
}
```

It meant when you create a `Reader[A, B]`, it gets callback function of A type and returns B type.

Example of creating a Reader:
```scala
case class Cat(sound: String)

// extracting the sound from the Cat class
val retrieveSound : Reader[Cat, String] = Reader {cat => 
  cat.sound
}

```

When you instantiate the Reader type, you pass in a callback function of the configuration that you are receiving, in this case `Cat`, and retrieve one of the attributes inside, in this case, `cat.sound`.

After you define the Reader type, you can call it by running the method `run` and supply the configuration as a parameter of `run`:
```scala 
// calling the cat
println(retrieveSound.run(Cat("meow"))) // meow
```

If you read until this point, you probably pause for a moment and think, "What does Reader type Monad so especially with just regular validation raw function?"

The answer, my friend, is that the Monad type is what makes it unique.

With being a Monad, the Reader type can call map and flatMap to chain operation, and modify the operation from an existing Reader.

Usually, you create a set of Reader that will accept the same type of configuration. Then, you can combine and chain them with flatMap and map, and then use `run` and supply that configuration at the end.

### map
The map method modifies the computation inside the Reader, bypassing the function to the result of the callback function.

```scala 
// from previous Cat example
val checkSound: Reader[Cat, Boolean] = retrieveSound.map(_ == "meow")

checkSound.run(Cat("bark")) // false
```

### flatMap
The flatMap operation is what makes Reader so powerful. It helps combine operations that depend on the same input type.
```scala
val greet:Reader[Cat, String] = Reader{cat =>
  s"hello $cat"
}

val sound:Reader[Cat, String] = Reader{cat =>
  s"${cat.sound} ${cat.sound}"
}

val greetAndSound = for{
  g <- greet
  check <- checkSound
  s <- sound
} yield {
  if(check) g + s else "sound is not right"
}

val result = greetAndSound.run(Cat("meow"))
println(result)
val notRight = greetAndSound.run(Cat("bark"))
println(notRight)
```

## Reader in Action
Reader can set up all your operations and inject your operation with a configuration as a parameter. 

Let's create a validation check function, given an id and an email, check whether there is an email corresponding to the id in the database. Let's create our Repository case class:
```scala
case class Repository(userDB:Map[Long, String], emailDB: Map[String, List[String]])

```

`userDB` maps id, 'Long`, to username,`String`. `emailDB` maps username to a list of email.

First, we need to create a Reader type getUser that gets an id and retrieve the username.
```scala
def getUser(id:Long):Reader[Repository, Option[String]] = Reader{repo =>
    repo.userDB.get(id)
  }
```

Then, we create getEmail which will get the username and return a list of email.
```scala 
def getEmail(username:Option[String]): Reader[Respository, List[String]] = Reader {repo =>
    repo.emailDB.getOrElse(username.fold("none")(st => st), List.empty[String])
  }
```

Lastly, we can have Reader that `checkIfEmailMatch` that takes in a parameter of id and email and check if the id contains the email address. With this method, we combine `getUser` method and `getEmail`. Then, we check if the List of emails matches the email that is input in the parameter.
```scala 
def checkIfEmailMatch(id:Long, email:String):UserReader[Boolean] = for {
    usernameOption <- getUser(id)
    emails <- getEmail(usernameOption)
  } yield {

    emails.contains(email)
  }
```

How do you execute `checkIfEmailMatch`?

Let's set up the main method:
```scala 
val userDB = Map(
    1L -> "john",
    2L -> "jane",
    3L -> "kate"
)
val emailDB = Map(
  "john" -> List("something@gmail.com"),
  "jane" -> List("jane@yahoo.com", "jane@gmail.com"),
  "kate" -> List("kate@hotmail.com", "kate123@yahoo.com")
)

val repo = Repository(userDB,emailDB)
val res = checkIfEmailMatch(2L, "jane@gmail.com").run(repo)
println(res)
```

We set up all the operations, started from `getUser`, then `getEmail`. We, then, combine the two operations in `checkIfEmail` exist. These all take in 1 configuration file, which is the repo. When you call the `run` method, it executes the operation.

## Takeaway
- Reader provides a tool for dependency injection - by setting up all the operation that takes in the same configuration.
- Readers are most useful when we want to construct a batch program that can easily represent as function; defer injection of a known parameter, and isolate the parts of the program that we want to test.
- By setting your system as readers, you can represent your readers as a pure function, and do map or flatMap for combining them.

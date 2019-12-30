---
layout: post
title: WTF is a Type Class?
date: 2019-12-29 20:57
summary: Learn how most author build their functional application with this technique
categories: functional-programming 
tags: functional-programming scala cats
---

![Photo by Cris DiNoto](https://images.unsplash.com/photo-1500856311637-fc0249e33e4c?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80)

I have a hard time understanding about type class a couple of weeks back while reading about <a href="https://books.underscore.io/scala-with-cats/scala-with-cats.pdf" target="_blank">Scala with Cats</a>. 

After countless research on the internet and other tutorial explaining Type Classes, I also decided to share my take and the way I look at them.

By knowing that the author of Cats Library used this technique to implement a lot of the functionality, I think it is handy as it was a widely used technique in designing the FP application. 

This technique does not only limited to FP, but OOP can also use this technique to create a more modular system.

Type class is an interface that defines some behavior. 

It is a way to create any behavior from an existing class without modifying anything on that source code. Instead, it creates a type class to implement the new behavior.


## Type class usually define in 3 things:
  - Trait Type Class
  - Instances of Type class: Defining the type class that we care about (concrete with implicit).
  - Interface objects or interface syntax

A simple example from Alvin Alexander <a href="https://alvinalexander.com/scala/fp-book/type-classes-101-introduction" target="_blank">blog</a>:
Let's say you have these existing data types:
```
sealed trait Animal
final case class Dog(name:String) extends Animal
final case class Cat(name:String) extends Animal
final case class Bird(name:String) extends Animal
```

Assumed that one day you need to add new behavior to Dog because it can `speak like humans`, you want to add a new `speak` behavior, but without really changing the source code you already have, and also not changing the behavior for `Cat` and `Bird`

1. Create a Trait that has a generic parameter:
```
trait SpeakBehavior[A] {
  def speak(a:A):Unit
}
```

2. Create the instances of the type class that you want to enhance.
```
object SpeakLikeHuman{
  implicit val dogSpeakLikeHuman: SpeakBehavior[Dog] = new SpeakBehavior[Dog] {
    def speak(a:Dog): Unit = {
      println(s"I'm a Dog, my name is ${dog.name})
    }
  }
}
```


3. Create an API for the consumers(caller) of this type class 

The caller will be calling like this:
```
BehavesLikeHuman.speak(aDog) // interface object
aDog.speak // interface syntax (using implicit class)
```

Interface Object (more explicit):
```
object BehavesLikeHuman {
  def speak[A](a:A)(implicit instance: SpeakLikeHuman[A]) = instance.speak(a)
}
```
To call this type class:
```
import SpeakLikeHuman._ // import all the implicits
val dog = Dog("Rover")
BehavesLikeHuman.speak(dog)
```

Interface Syntax (implicit):
```
object BehavesLikeHumanSyntax {
  implicit class BehavesLikeHumanOps[A](a:A) {
    def speak(implicit instance:SpeakLikeHuman[A]): Unit = {
      instance.speak(a)
    }
  }
}
```

Calling this:
```
import SpeakLikeHuman._ // import the instances
import BehavesLikeHumanSyntax.BehavesLikeHumanOps

val dog = Dog("Rover")
dog.speak // because implicitly gets it from the implicit class BehavesLikeHumanOps
```

To me, calling Interface syntax is not as readable as calling Interface objects, especially when you are trying to read a large codebase. 

Implicit in Scala can be a double-edged sword, and for someone who is not used to reading Scala code, it can be hard for the first time to read a large codebase with multiple implicit. However, if you are using interface syntax on your API, the caller can invoke a new behavior directly on the instance - it looks like you added a new behavior to the `Dog` instance without changing any of the source code.

## 3 Main Takeaways
- Type Class is like inheritance, but in an FP way, you get to create new behavior on the model without modifying the existing source code.
- There are 3 steps to create a Type Class - the Interface, the Type-class instance, and the API
- Be aware of interface syntax, and implicits in Scala because sometimes it can be hard to debug if you used it too often in a large codebase.

---
layout: post
title: What is an ADT (Algebraic Data types)?
date: 2019-12-29 20:57
summary: Chances are, you already did it when you build your application even you didn't know what it is
categories: functional-programming
tags: functional-programming tech scala software-development
---

![Photo by John Moeses Bauan](https://images.unsplash.com/photo-1528082992860-d520150d6f6c?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2468&q=80)


I got a <a href="http://www.agilemodeling.com/artifacts/userStory.htm" target="_blank"> story</a> to research on whether <a href="https://circe.github.io/circe/codecs/semiauto-derivation.html" target="_blank">Circe</a> can polymorphically parse Json objects to ADT. One of the questions that I got was, what is an ADT? 

I murmured for a couple of minutes - realizing that I could not say a concrete explanation on what is ADT. 

After being called out in front of the standups, I decided that I have to understand what Algebraic Data Types is - and I think this knowledge is essential and is often used in a lot of Functional Programming applications.

To Quote <a href="http://alvinalexander.com/" target="_blank">Alvin Alexander</a> - 

> Algebraic data types are not a way of proactively designing FP (Functional Programming) code; instead, they are a way of categorizing data models you've already written, especially FP data models. 

We usually know ADT in a lot of FP data models like Enumeration (Enum) type in OOP. However, ADT represents a broader category than just `Enums`.

One of the advantages of writing ADT:
1.  is that you can do pattern matching style of programming.
2. Your code becomes more "algebra"


## What does "algebra" means?
A lot of us know algebra that we encounter in high school.
Example: 
```
1+1 = 2
```

Those are numeric algebra.

However, as you study more about FP, you get to understand that algebra consists of 2 characteristics:
1. A set of objects
2. Operations that applied to those objects to create new objects

In the numeric algebra case, for instance, `1` are a set of objects of numerical types, and the operators are `+`. Therefore, `2` is consists of `1` numerical types that applied with an operators `+`.

There are various "algebra" besides, numerical algebra. There is relational algebra, which is used in the database.  In relational algebra, the table represents the set of objects, and operators such as `SELECT`, `AS` are represented as the operators. You can see more types of algebra <a href="https://en.wikipedia.org/wiki/Algebra" target="_blank">here</a>.

### How is algebra works in programming?
```
case class Coordinates(x:Int, y:Int)
```
Chances are, you have already work on ADT even if you don't know it. The example above represents algebra in programming, because of a set of objects (integers of x, and y), and the constructor's operators (case class) that creates a new type (Coordinates)


## There are 3 types of ADT in scala:
### Sum Types - usually with sealed trait and a case objects.
Sum types also called enumeration type because you enumerate all the possible instances from the base type.
Usually, you phrase those types with "is a" and "or"

### Product Types - case class
Product types are a data type that you create with case class, that has a parameter. Instead of creating a data type whose number of possibilities of concrete instance that you list that extends the base type, it determines the number of possibilities that you have in your constructor fields. 
Usually, you phrase said those types with a "have a" and "and"

Example:

```
case class Coordinates(x:Int, y:Int)

```
How many combinations that need to account for this one?

In this case, it is 2 * 2^256 because there are that many possibilities in an integer.

### Hybrid - both sum and product type, a sealed trait with a case class
Hybrid types follow a sealed trait and a case class.
```
sealed trait PaymentInstrument
case class Paypal(accountId:String, amount:Int)
case class Ideal(accountId:String, amount:Int)
```
In this case, `Paypal` `is-a` type of `PaymentInstrument`, or you can phrase it as `PaymentInstrument` is a `Paypal` `or` `Ideal`.

Then Paypal is a product type where Paypal `has-a` `accountId` `and` `amount`.


As you noticed, if a case class except a String, there are an infinite amount of possibilities. If a class has an `Int`, you need to account for 256, and if it accepts a boolean, you need to account for 2 possibilities. Therefore, the fewer possibilities you have to deal with, the amount o code you have to write. One scenario that you should think about when designing your data types.

## 3 Main Takeaways
- Algebraic Data Types is not a way to design your functional programming code, but to _categories_ your data model.
- There are 3 cases of Algebraic Data Types (Sum, Product, and Hybrid)
- After understanding ADT, you also get to realized - more parameters and combination you create in your data types equal more cases that you need to cover. If you need to cover more cases in your code, the unit test is much harder.
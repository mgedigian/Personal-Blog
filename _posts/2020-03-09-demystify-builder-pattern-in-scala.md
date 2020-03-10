---
layout: post
title: Demystify Builder Pattern in Scala
date: 2020-03-09 22:46
summary: It is surprisingly simple to create a Builder Pattern in Scala
categories: scala software-development programming
tags: scala software-development programming
---


![Photo by Chris Gray](https://images.unsplash.com/photo-1542350880924-09225f70e026?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=666&q=80)

The builder pattern solves the issue when you need to construct a complicated object step by step without complicating the construction code. 

Imagine designing an HTTP handler, where you need to construct an HTTP client that takes in URL, various headers, and also a payload body depending on what sort of request you want. 

```scala
val postRequestClient = HttpClientRequest(endpoint = "https://someURL.com", method = "POST", header = Map("content-type" -> "application/json"), body = "your json body")

// send your post request
HttpClient.send(postRequestClient)

```

You can design the constructor of the above `HttpClient` as this:

```scala
class HttpClientRequest(endpoint:String, method:String, header:Map[String, String], body:Option[String]) 
```

Then, you also need to account for the get method where there is no `requestBody`.

```scala
// inside HttpClientRequest class

// add another constructor
def this(endPoint:String header:Map[String,String]) {
  HttpClientRequest(endpoint, "GET", header, None)
}

```

As you can imagine, step by step of the laborious process of initializing many fields and nested objects. The code of creating such initialization is monstrous complex and redundant - with various combinations of constructor fields. If the size of the new constructor parameter increases, so does several combinations of your constructor in the class.

How are we going to solve this issue?


## Builder Pattern to the Rescue
The builder pattern suggests that you can abstract out the initialization pattern from the regular constructor to the builder class.

The `builder` class makes the creation of this object into a series of steps. Therefore, when you want to initialize this object, you execute a series of steps in the builder object.

The important part is that when the callers execute the series of steps in object creation, they don't need to call all of the steps. The callers essential to execute the steps that are necessary for the object.

Let's refactor the `HttpClient` example above with the builder class - called `HttpClientBuilder`.

Note: Scala provided an elegant solution in the problem above by using an immutable `case class` with default argument and named parameters.


```scala
class HttpClientRequest(endpoint:String, method:String, header: Map[String,String], body:Option[String])

object HttpClientRequest {
  def builder(): HttpClientRequestBuilder = HttpClientRequestBuilder("")
}

// creating method case class
sealed trait Method
case object GET extends method
case object POST extends method
case object PUT extends method
case object DELETE extends method
case object PATCH extends Method

class HttpClientRequestBuilder(endpoint:String, method:Method = GET, header: Map[String,String] = Map.empty[String,String], body:String = "") {
  def withEndPoint(endpoint:String): HttpClientRequestBuilder = copy(endpoint = endpoint)
  
  def withMethod(method:Method): HttpClientRequestBuilder = copy(method = method)
  
  def withHeader(headers:Map[String,String]): HttpClientRequestBuilder = copy(header = headers)
  
  def withBody(body:String): HttpClientRequestBuilder = copy(body = body)
  
  
  def build: HttpClientRequest = new HttpClientRequest(endpoint, method, header)
}

```

I created a sealed trait so that it restricts the caller to the method. 

Scala `case class` has already have a built-in method `copy` for you, which creates a deep copy of the current object to a new object with a different object that specifies in the argument.

The HttpClientBuilder create a series of steps in the initialization of `HttpClient`.

Now, you can create the HttpClient method like this:
```scala

val postClientRequest = HttpClient.builder.withEndPoint("https://someURL.com").withMethod(POST)
  .withHeader(Map("content-type" -> "application/json")).withBody("json body ... ").build

HttpClient.send(postClientRequest)

```

## Takeaway
Builder pattern lets you construct a complex object in a step by step fashion so that you can create various representations of the object through the same construction code.

Builder pattern uses the builder class to make initialization as a series of steps to execute. The most important thing about the builder class is that the caller able to provide only the necessary steps that are needed to configure the object.

Scala provided an immutable case class and a combination of default arguments and named parameter, which is reaching the same purpose of the builder pattern.

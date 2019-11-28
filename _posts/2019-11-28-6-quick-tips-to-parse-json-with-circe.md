---
layout: post
title: 6 quick tips to Parse JSON with Circe
date: 2019-11-28 14:17
summary: All you question about Circe will be answer in here
categories: scala tech soft-development ETL Circe
tags: scala tech soft-development ETL Circe
---

<img src="{{site.baseurl}}/images/6-quick-tips-to-parse-json-with-circe/6 QUICK tips to parse Circe.png" alt="6 QUICK tips to parse Circe">

Marshall and Unmarhsall JSON is the bread and butter of ETL. There has been much library that gives ways to help the data wrangling process much seamless. One of these JSON libraries that gain much popularity lately is Circe. It is a library that creates a shapeless dependency that has an automatic deserialization function that serializes JSON string to a domain model. However, these JSON libraries also come with a catch. For example, you keep encountering decode failure if you don't know what you need to use the `prepare` method to handle class with default on the non-optional field. Decoding nested arrays and objects is hard if you don't know some vital work around the libraries. You encounter a lot of questions, such as what is the difference between auto and semi-auto, and when do you need to use one vs. the other? 

These questions can sometimes be daunting. I spent days and weeks, scratching my head, pulling my hair, understand and learn all the best practices in using this JSON library – I  concluded 6 simple tips that I learn that can save you tons of time in using Circe in your future Scala projects.

## Setup
Before we started, let's setup up the environment. If you haven't set up SBT, you can look at the documentation <a href="https://www.scala-sbt.org/1.x/docs/" target="_blank">here</a>. Paste the dependency on your `build.sbt` file:

```
val circeVersion = "0.11.1"

libraryDependencies ++= Seq(
  "io.circe" %% "circe-core",
  "io.circe" %% "circe-generic",
  "io.circe" %% "circe-parser"
).map(_ % circeVersion)
```


Let's get down to 6 tips and the examples on ways to decode JSON in Scala:

## Decoding Objects
{% highlight java %}
case class Product(productId: Long, price: Double, countryCurrency: String, inStock: Boolean)

object Product {
  implicit val decoder: Decoder[Product] = deriveDecoder[Product]
  implicit val encoder: Encoder[Product] = deriveEncoder[Product]

  def main(args: Array[String]): Unit = {
    val inputString: String =
      """
        |{
        |   "productId": 111112222222,
        |   "price": 23.45,
        |   "countryCurrency": "USD",
        |   "inStock": true
        |}
        |""".stripMargin

    decode[Product](inputString) match {
      case Right(productObject) => println(productObject)
      case Left(ex) => println(s"Ooops some errror here ${ex}")
    }
  }
}

{% endhighlight %}

Define the domain model for decoding the JSON value by defining an implicit encode and decode value with Circe. 

Noticed the `implicit Val` in objects? When you define an implicit value in the companion objects (implicit scope), you don't need to introduce the import tax when you call the `decode[A]` value.

## Decoding Arrays
{% highlight java %}
case class Book(book: String)

object Book {
  implicit val decoder: Decoder[Book] = deriveDecoder[Book]

  def main(args: Array[String]): Unit = {
    val inputString =
      """
        |[
        | {"book": "Programming in Scala"},
        | {"book": "How to Win Friends and Influence People"},
        | {"book": "HomoSapiens"},
        | {"book": "Scala OOP"}
        |]
        |""".stripMargin

    parser.decode[List[Book]](inputString) match {
      case Right(books) => println(s"Here are the books ${books}")
      case Left(ex) => println(s"Ooops something error ${ex}")
    }
  }
}

{% endhighlight %}

When you decode a list of arrays, set the case class to a single object.  Then, when you invoke decode parser, you need to specify what data structure is holding that JSON value – in this case, it is a `List`: `parser.decode[List[Book]]`.

## Decoding Automatically
{% highlight java %}
case class Form(firstName: String, lastName: String, age: Int, email: Option[String])

object Form {

  def main(args: Array[String]): Unit = {
    val inputString =
      """
        |[
        |    {"firstName": "Rose", "lastName":"Jane", "age":20, "email":"roseJane@gmail.com"},
        |    {"firstName": "John", "lastName":"Doe" , "age": 45}
        |]
        |""".stripMargin

    parser.decode[List[Form]](inputString) match {
      case Right(form) => println(form)
      case Left(ex) => println(s"Ooops something happened ${ex}")
    }
  }
}

{% endhighlight %}

Circe has an auto decoder feature that automatically decodes a string of JSON to the domain model for you. You need to import `io.generic.auto._`

## Decoding Manually
{% highlight java %}
case class Applicant(name: String, age: Int, phoneNumber: String)

object Applicant {
  // here we are actually casting the return value to Decode
  implicit val decoder: Decoder[Applicant] = (hCursor: HCursor) =>
    for {
      name <- hCursor.get[String]("name")
      age <- hCursor.downField("age").as[Int]
      phoneNumber <- hCursor.get[String]("phoneNumber")
    } yield Applicant(name, age, phoneNumber)

  def main(args: Array[String]): Unit = {
    val inputString =
      """
        |[
        | {"name": "Jane Doe", "age":26, "phoneNumber":"512222222"},
        | {"name": "Petter Pan", "age":55, "phoneNumber":"214553356"},
        | {"name": "Jason Mamoa", "age":33, "phoneNumber":"2111112234", "email":"jasonMamoa@gmail.com"}
        |]
        |""".stripMargin

    parser.decode[List[Applicant]](inputString) match {
      case Right(applicants) => println(applicants)
      case Left(ex) => println(s"Oops something is wrong with decoding value ${ex}")
    }
  }
}

{% endhighlight %}

#### What do we learn here?
- You can write your decode with `HCursor`. 
- If you use the above syntax by calling anonymous function `(hcursor:HCursor)`, then you need to cast the return value to a `Decoder[A]`.
- `.get[A](key)`  with `.downfield("key").as[A]` is the same thing.
- If you have multiple nested JSON, specifying the type as the type that you want to decode, Circe recursively decodes the next value for you.

## Decoding an Arrays of Objects of Arrays

Having multiple nested JSON String can be tricky and hard to decode. However, with a little bit of help from <a href="https://typelevel.org/cats/" target="_blank">Cats</a>, you can retrieve values from an array of objects of arrays.

Assume there is an incoming JSON string like this:

```
[
  { "name":"productResource", orderItems: [ { voucher: { "campaignNumber":12, "discount":20, "subscriptionPeriod" "June" }}, { voucher: { "campaignNumber":13, "discount":24 }}] },
  { "name":"productResource2", orderItems: [ { voucher: { "campaignNumber":13, "discount":24 }}] },
  { "name":"productResource3", orderItems: [ { voucher: { "campaignNumber":15, "discount":28 }}] }
]
```

This is a `priceService` object. 

Let's say you want to get an array of `campaignNumber` which is 2 levels deep. You want to assign `campaignNumber` into a different model name.

With a little help from `HCursor` and a little help of Cats Traverse, you can retrieve `campaignNumber`. Like this:

{% highlight java %}

case class ProductResrsource(name: String, campaignResources: List[Int], discountPrice: List[Int])

object voucher {
  implicit val decoder: Decoder[ProductResrsource] = new Decoder[ProductResrsource] {
    override def apply(hCursor: HCursor): Result[ProductResrsource] =
      for {
        name <- hCursor.downField("name").as[String]
        orderItemsJson <- hCursor.downField("orderItems").as[List[Json]]
        campaignResource <- Traverse[List].traverse(orderItemsJson)(
          itemJson => itemJson.hcursor.downField("voucher").downField("campaignNumber").as[Int]
        )
        discountPrice <- Traverse[List].traverse(orderItemsJson)(orderItemsJson => {
          orderItemsJson.hcursor.downField("voucher").downField("discount").as[Int]
        })
      } yield {
        ProductResrsource(name, campaignResource, discountPrice)
      }
  }

  def main(args: Array[String]): Unit = {
    val inputString =
      """
        |[
        |   {
        |      "name":"productResource",
        |      "orderItems":[
        |         {
        |            "voucher":{
        |               "campaignNumber":12,
        |               "discount":20,
        |               "subscriptionPeriod":"June"
        |            }
        |         },
        |         {
        |            "voucher":{
        |               "campaignNumber":13,
        |               "discount":24
        |            }
        |         }
        |      ]
        |   },
        |   {
        |      "name":"productResource2",
        |      "orderItems":[
        |         {
        |            "voucher":{
        |               "campaignNumber":13,
        |               "discount":24
        |            }
        |         }
        |      ]
        |   },
        |   {
        |      "name":"productResource3",
        |      "orderItems":[
        |         {
        |            "voucher":{
        |               "campaignNumber":15,
        |               "discount":28
        |            }
        |         }
        |      ]
        |   }
        |]
        |""".stripMargin
    parser.decode[List[ProductResrsource]](inputString) match {
      case Right(vouchers) => vouchers.map(println)
      case Left(ex) => println(s"Something wrong ${ex}")
    }
  }
}
{% endhighlight %}

#### What do we learn here?
- If there is a nested array and you want to traverse the next nested array, extract that top-level to a `List[JSON]` and use cats `Traverse` method.
- All items wrap inside the for comprehension will return  `Either[DecodeFailure,A]`. Since the items are wrapped in a List, it will return  `List[Either[DecodeFailure,A]]`.
- Cats `Traverse` helps flip `List[Either]` to `Either[List]`.
- All JSON objects can be traverse with `HCursor`, so use HCursor to traverse the nested JSON string to get the attributes `campaignNumber`.

## Handling Class with Default on Non-Optional Field
If you create a domain model with a default argument, there be no problem if the caller doesn't provide that argument. However, trying to decode that value in Circe throws decoder failure - Circe is not able to decode the non-optional field if you don't put optional on that field.

But what if you don't want to wrap the argument in an `Option` because it is not optional?

The answer is to use `prepare`. 

`prepare` helps you modify the JSON before Circe decodes it so that it doesn't throw any error.

Let's say in this example, not all the companies that provided data have a `public` flag. 

```
[
 {"industry":"tech", "year":1990, "name":"Intel", "public": true},
 {"industry":"tech", "year":2006, "name":"Netflix"},
 {"industry":"Consumer Goods", "year":1860, "name":"Pepsoden", "public": true}
]
```

Therefore, we need to set the `public` flag to false if it doesn't exist. 

{% highlight java %}
case class Company(industry: String, year: Int, name: String, public: Boolean)

object Job {
  implicit val decoder: Decoder[Company] = deriveDecoder[Company].prepare { (aCursor: ACursor) =>
    {
      aCursor.withFocus(json => {
        json.mapObject(jsonObject => {
          if (jsonObject.contains("public")) {
            jsonObject
          } else {
            jsonObject.add("public", Json.fromBoolean(false))
          }
        })
      })
    }
  }

  def main(args: Array[String]): Unit = {
    val inputString =
      """
        |[
        | {"industry":"tech", "year":1990, "name":"Intel", "public": true},
        | {"industry":"tech", "year":2006, "name":"Netflix"},
        | {"industry":"Consumer Goods", "year":1860, "name":"Pepsoden", "public": true}
        |]
        |""".stripMargin

    parser.decode[List[Company]](inputString) match {
      case Right(companies) => companies.map(println)
      case Left(ex) => println(s"ooops something wrong ${ex}")
    }
  }
}
{% endhighlight %}

And That's it!

## In Summary:
There are 3 ways to decode JSON with Circe – auto, semi-auto, and manual.

Circe can automatically decode standard type containers such as List or Option.

You don't need to decode all the values in the JSON file, you can use `deriveDecode,` and it automatically decodes all levels of JSON string for you.

If you want to deserialize multiple nested objects of arrays, you can use Cats `Traverse` to get nested attributes.

When you want to preprocess JSON string before letting Circe decodes it for you, use `prepare`.

I hope this post can help you solved any confusion about Circe, and any feedback is welcomed. If there are any other gotchas that you find in parsing JSON with Circe, please comment it below.

The full source code of this tutorial is <a href="https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/parsejsonwithcircetutorial/src/main/scala" target="_blank">here</a>.

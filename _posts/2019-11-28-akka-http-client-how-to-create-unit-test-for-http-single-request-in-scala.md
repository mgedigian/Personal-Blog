---
layout: post
title: Akka Http Client- How to create Unit test for Http Single Request in Scala
date: 2019-11-28 14:17
summary: Follow this process and your unit test will be breeze
categories: unit-test software-development scala
tags: unit-test software-development scala
---

<img src="{{site.baseurl}}/images/akka-http-client-how-to-create-unit-test-for-http-single-request-in-scala/Unit-Test-Akka-Http.png" alt="unit-test-akka-http">

Unit testing is often discussed in software development. When it comes to writing functions, we tend to think about how we can integrate that function in the entry point of the application. When it comes to unit testing that function, having all branch coverage inside that function can be hard and tricky.

Akka HTTP has a helpful testkit to test routes that make unit testing simple. Testing without TCP connection should not be complicated. You can mock the IO function, such as `singleRequest`.  The key to a readable and accessible test suite lies in how you structure the functions.

In this tutorial, I want to share how you can structure the Http `singleRequest` function in Akka HTTP so that it is easier to mock and get 100% branch coverage.

## The Problem
In this tutorial, I use a fake response of a simple dummy rest-client which retrieves dummy employee information. The task is to create a client service that gets the employee information through Http SingleRequest and parse all the fields in the response. Usually, we write a function like this:
{% highlight java %}
def getEmployees(url: String): Future[List[Employee]] = {
    for {
      response <- http().singleRequest(HttpRequest(uri = url))
      employeeString <- Unmarshal(response.entity).to[String]
      employeeObject <- parser.decode[List[Employee]](employeeString) match {
        case Right(employeeObj) => Future(employeeObj)
        case Left(err) => Future.failed(err)
      }
    } yield { employeeObject }
{% endhighlight %}

The function creates a fetch call through `http().singleRequest` and deserialize the JSON string to an Employee model class with <a href="https://circe.github.io/circe/" target="_blank">Circe</a>.

This function becomes hard for unit testing because the IO function coupled with other operations in the function.

## The Intuition
To make the application as loosely coupled as possible. We can wrap the IO function into a trait so that the functionality can be overridden or mocked.

In this case, `http().singleRequest` is the component that creates IO. Therefore, we can extract the functionality out and wrap it into a trait.

{% highlight java %}
trait HttpClient {
  def sendRequest(httpRequest: HttpRequest)(implicit actorSystem: ActorSystem): Future[HttpResponse]
}
{% endhighlight %}

Because we extracted the `singleRequest` as an abstract function, we can mock the function in the unit test with Scalamock.

Order of Execution
We can write the regular implementation first; then we can refactor the function by extracting the `http().singleRequest`.

We will use Circe to encode and decode JSON String, and Scalamock to mock the IO functions.

Let's create a domain model Employee:

{% highlight java %}
case class Employee(id: String, employeeName: String, employeeSalary: String, employeeAge: String, profileImage: String)
{% endhighlight %}

Once we create a domain model, let's create the main class. Writing the main class helps you design how you want to write the function. It also helps to understand how another caller uses your function. Therefore, the main class is a simple way to test the function. 

{% highlight java %}
object EmployeeRestClient extends App {
  implicit def actorSystem: ActorSystem = ActorSystem()
  implicit val executionContext: ExecutionContext = actorSystem.dispatcher

  val newRestClient = new EmployeeRestClient() with ClientHandler {
    override implicit def actorSystem: ActorSystem = ActorSystem()
    override implicit def executionContext: ExecutionContext = actorSystem.dispatcher
  }
  newRestClient.getEmployees("http://dummy.restapiexample.com/api/v1/employees").onComplete { res =>
    res match {
      case Success(employees) => employees.map(println)
      case Failure(exception) => println(s"Failed to fetch ... ${exception.getMessage}")
    }
    newRestClient.shutDown()
  }
}
{% endhighlight %}


Inside this function, we extract out the fetch endpoint URL by wrapping it with the `getEmployee` function. 

{% highlight java %}
newRestClient.getEmployees("http://dummy.restapiexample.com/api/v1/employees").onComplete { res =>
    res match {
      case Success(employees) => employees.map(println)
      case Failure(exception) => println(s"Failed to fetch ... ${exception.getMessage}")
    }
{% endhighlight %}

{% highlight java %}
def getEmployees(url: String): Future[List[Employee]] = {
    for {
      response <- http().singleRequest(HttpRequest(uri = url))
      employeeString <- Unmarshal(response.entity).to[String]
      employeeObject <- parser.decode[List[Employee]](employeeString) match {
        case Right(employeeObj) => Future(employeeObj)
        case Left(err) => Future.failed(err)
      }
    } yield { employeeObject }
  }
{% endhighlight %}


## The Refactor
Let's refactor the code so that `handleRequest` can be mock.

How?

Let's create a trait, HttpClient, that has an abstract function `sendRequest`. Then, let's mock `sendRequest` later in the unit test.

{% highlight java %}
trait HttpClient {
  def sendRequest(httpRequest: HttpRequest)(implicit actorSystem: ActorSystem): Future[HttpResponse]
}
{% endhighlight %}

Then, let's use self-type annotation in the `EmployeeRestClient` trait so that any instantiation or any class that extends the `EmployeeRestClient` also need to extend the HttpClient trait. This is very useful for later when we instantiate the service.

{% highlight java %}
trait EmployeeRestClient { this: HttpClient =>
  implicit def actorSystem: ActorSystem
  implicit def executionContext: ExecutionContext

  def getEmployees(url: String): Future[List[Employee]] = {
    for {
      response <- sendRequest(HttpRequest(uri = url))
      employeeString <- Unmarshal(response.entity).to[String]
      employeeObject <- parser.decode[List[Employee]](employeeString) match {
        case Right(employeeObj) => Future(employeeObj)
        case Left(err) => Future.failed(err)
      }
    } yield { employeeObject }
  }
}
{% endhighlight %}

Then, we create `ClientHandler` trait which contains the `http().singleRequest` Function. We will not test this trait, but this trait will be mocked in the unit test later. 

{% highlight java %}
trait ClientHandler extends HttpClient {
  override def sendRequest(httpRequest: HttpRequest)(implicit actorSystem: ActorSystem): Future[HttpResponse] = {
    Http().singleRequest(httpRequest)
  }

  def shutDown()(implicit actorSystem: ActorSystem): Unit = {
    Http().shutdownAllConnectionPools()
  }
}
{% endhighlight %}

This refactor separated JSON deserialization and `singleRequest`. Then, we can test `EmployeeRestClient` trait `getEmployees`, by mocking the function on `sendRequest`.

## Testing
Let's set up all the mocks. 

Since we extract `httpRequest`, we can create another trait that also extends `HttpClient` and mock the function. 

{% highlight java %}
 trait MockClientHandler extends HttpClient {
    val mock = mockFunction[HttpRequest, Future[HttpResponse]]

    override def sendRequest(httpRequest: HttpRequest)(implicit actorSystem: ActorSystem): Future[HttpResponse] =
      mock(httpRequest)
  }
{% endhighlight %}

You can specify the expected input and returns on the function. 

<href="https://scalamock.org/" target="_blank">Scalamock</a> automatically mock the function for you. Therefore, no side effect or IO involved in running the `getEmployee`.

{% highlight java %}
employeeRestClient.mock
.expects(HttpRequest(uri = "http://dummy.restapiexample.com/api/v1/employees"))
.returning(Future.successful(HttpResponse(entity = HttpEntity(ByteString(stripString)))))

{% endhighlight %}


Once we mocked the function, we start setting up the test suite. The test suite involves a setup environment, mock expectation, and assertion. Once we create an expectation for the function in ScalaMock, it also creates an assertion. If there is a discrepancy between the function call and the expectation, that test suite fails. 

Did you notice that the test suite is precisely like the Main function that we defined earlier? 

Starting from the Main function makes it easy to construct the test suite. 

The difference between the main and the test suite lies in the different `ClientHandler` traits. In main, we extend `EmployeeRestClient` with `ClientHandler`. In the test suite, we mock the `ClientHandler`. 

By doing this, we can test `getEmployee` functionality because we wrap the `http().singleRequest` separately into another function.

{% highlight java %}
"Employee Rest Client" should {
    "work on HttpClient" in {
      val stripString =
        """
          |[{
          |"id": "1",
          |"employee_name": "Salome",
          |"employee_salary": "457",
          |"employee_age": "35",
          |"profile_image": ""
          |},
          |{
          |"id": "34",
          |"employee_name": "ABC",
          |"employee_salary": "666",
          |"employee_age": "50",
          |"profile_image": ""
          |}]
          |""".stripMargin

      // mock Http
      val employeeRestClient = new EmployeeRestClient with MockClientHandler {
        override implicit def actorSystem: ActorSystem = system
        override implicit def executionContext: ExecutionContext = ExecutionContext.Implicits.global
      }

      employeeRestClient.mock
        .expects(HttpRequest(uri = "http://dummy.restapiexample.com/api/v1/employees"))
        .returning(Future.successful(HttpResponse(entity = HttpEntity(ByteString(stripString)))))

      val expectResult = parser.decode[List[Employee]](stripString) match {
        case Right(employees) => employees
        case Left(_) => throw new Exception
      }

      whenReady(employeeRestClient.getEmployees("http://dummy.restapiexample.com/api/v1/employees")) { res =>
        res must equal(expectResult)
      }
    }
  }
{% endhighlight %}

## In Conclusion:
- Design your function from the main class
- Refactor your design by extract the IO portion of the function for mocking later.
- Mock the extracted IO function with any mock library you want, in this case, Scalamock.
- Construct your test suite like how you would call your function from the main class


That's it! I hope this helps you write better unit test in your Scala projects or unit test `HttpClient` request. The full implementation of this tutorial is on <a href="https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/testingakkahttptutorial/src" target="_blank">GitHub</a>. Please comment below if there are any questions and comments. If you have any other great strategies, feel free to share your knowledge in the comment section below!

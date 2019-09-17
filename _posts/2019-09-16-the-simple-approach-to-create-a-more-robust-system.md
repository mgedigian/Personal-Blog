---
layout: post
title: The Simple Approach to Create a More Robust System
date: 2019-09-16 21:46
summary: All system will failed, but this approach can help you prevent bigger problem in the future.
categories: tech software-development
tags: software-development tech best-practice
---

![Photo by Adli Wahid](https://images.unsplash.com/photo-1542828810-3372a0020f50?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80)

*"System should not fail, and application should not crash."* This mantra has been widely implemented in software development and designing a distributed system. Various company has implemented some sort of methods to prevent their application for crashing. For instance, Amazon protects its cloud infrastructure from crashing by using "<a target="_blank" href="https://www.inc.com/kevin-j-ryan/the-crazy-ways-amazon-will-prevent-your-site-from-crashing.html">highly available system</a>", which refers to the system without a single point of failure – through creating redundancy so that if one component fails, the system can remain in operation. Facebook uses partitioning its MYSQL database to each region when first started to prevent their application to crash. However, just like all human make mistake, all system will encounter failure. There are various ways to limit the application for not crashing, such delaying the failure, finding a workaround the bugs so that the system can keep running, and just failed it silently. These methods are not the best way to handle system failure.

Why? Because doing those three things didn't solve the problem. The bug still exists. Hiding the problem will just cause the problem to get bigger in the future that it will cost more time and resources to fix it.  

__So, what should we do if the application fails?__

The answer is to fail fast, and then return early. We need to change how we approach software development. We should give exception and return it if there is any problem, instead of avoiding any sort of exception in a method and return gracefully.

<blockquote>
    <p>"The most annoying aspect of software development, for me, is debugging. The bug that I hate are the ones that showed up after the hourly successful operation, under unusual circumstances."</p>
    <footer><cite title="Jim Shore"> Jim Shore</cite></footer>
</blockquote>


## Fail fast, instead of Fail-Safe
When the software failed fast, it won't decrease the amount of bug in the system; but it will be easier to catch the bug and fix the problem. Fail fast will encourage us to fail the system instantly and visibly. It seemed like it will make our system fragile by exposing error immediately. However, it creates a more robust system that fixes the problem before it goes to production. When bugs are easier to detect and reproduce, it is easier to fix. For example, let's create a fail-safe method to calculate the amount of rent:
{% highlight java %}
public int maxRent() {
  RentPortfolio rent = Portfolio.get("rent-portfolio");
  if(rent == null) {
    return 1000;
  } 
  return rent.getMaxPrice();
}
{% endhighlight %}
This method calculates the maximum rent by getting the `rent-portfolio` property from the `Portfolio` object. When you feed in any value in this method, it will return gracefully. 

Now, imagine there is a minor upgrade in the `Portfolio` class, that one developer that is upgrading the code base creates a single typo on `rent-portfolio` to `rent-porttfolio`. The software will still be shipped successfully, and the application will still work. However, the user will get the wrong value each time since it will return a default value. When the client realized that it is not accurate, it will be hard to debug the issue.

Instead, if you write a fail-fast method:
{% highlight java %}
public int maxRent() {
  RentPortfolio rent = Portfolio.get("rent-portfolio");
  if(rent == null) {
    throw new PortfolioNotFoundException("rent-portfolio is not found in " + this.portfolioPath);
  }
  return rent.getMaxPrice();
}
{% endhighlight %}

This time, the developer will detect the bug before shipping the software because it throws a `PortfolioNotFoundException`.

Another example of fail-fast and fail-safe method:

***Fail-Safe***

{% highlight java %}
public division(int numerator, int denominator) {
  /* some code here ........ */
  return numerator / denominator;
}

public static void main(String...args) {
  division(2,0);
}
{% endhighlight %}
<span class="red">Exception: java.lang.ArithmeticException: / by zero ...</span>

***Fail-fast***
{% highlight java %}
public division(int numerator, int denominator) {
  if(denominator == 0) throw new IllegalArgumentException("denominator cannot be 0");
  /* some code here ..... */
  return numerator / denominator;
}

public static void main(String...args) {
  division(2,0);
}
{% endhighlight %}

<span class="red">Exception: java.lang.IllegalArgumentException: denominator cannot be 0 ....</span>

In this example, fail-fast does throw an error. However, the application is not throwing the right message. The root cause of the bug is because it is `InvalidArgumentException` instead of `ArithmeticException`. Thus, the developer needs to search through the code several times to know where the problem lies. Fail-fast will help the developer who maintains the program spot the bug immediately.

### Tips - A couple Fails Fast checks:
* Null
* Empty String
* Empty List
* Most pre-condition checks

### Ways of identifying visible failure:
* Using test-driven development. you will be able to state the success use-case and failed use-case by using assertion even before you implement your method. 
* Having a continuous integration build in your software development process will ensure any necessary failure when merging feature branch to master branch.
* Creating an assertion for a sanity check - the difference between assertion and exception can be found here (link).
* When there is any problem with system setup, instead of having a fall back function, failed the system so that you can be notified and fix the problem immediately.
* When the frontend requests a GET call for a collection to the server, instead of returning an empty list when it is not found, return a 404 not found exception.

## Return Early
Return early goes hand in hand with failed fast. When you discover a bug, you can either throw that to the system stack above the function or return that error immediately. For instance, when you want to create a pre-condition checks in your function parameters, you might do something like this:

{% highlight java %}
public String getPersonalData(Person person) {
  if(!systemIsUp) {
   if(person.getName() == "") {
    if(person.age == 0) {
      if(person.pin == "") {
        return "Person doesn't have a pin"; 
      }
      return "Person age cannot be 0";
    }
    return "Person Doesn't have a name"
   }
   return "System is Down"
  }
  
  return person.getName();
}
{% endhighlight %}

If you are reading this code, by the time you get to the third statement, you might forget what the statement of the previous two arguments needs to be. The main problem with this check is that not only it is hard to debug when something fails, but also hard to read. It leads to high cyclomatic complexity. <a target="_blank" href="https://en.wikipedia.org/wiki/Cyclomatic_complexity">Cyclomatic complexity</a> is a metrics that measure how complex your software system has. How do you measure the complexity? Whenever there is `if`, `else`, `try`, `catch`, `for`, `and`, `or`, `others`, `while`, you are adding a bit of complexity. Higher in Cyclomatic complexity means the system is complex, hard to maintain. Further, keeping track of all the checks and nested if/else statement is hard to keep in your memory. Instead, return the pre-conditioned if/else statement early, starting from the simple condition first. 


{% highlight java %}
public String getPersonal(Person person) {
  if(!systemIsUp) return "System is Down";
  
  if(person.getName() == "") return "Person Doesn't have a name";
  
  if(person.age == 0) return "Person age cannot be 0";
  
  if(person.pin == 0) return "Person' doesn't have a pin";
  
  return person.getName();
} 
{% endhighlight %}

If you are reading this code, you will understand what the pre-condition should be. You don't need to have a piece of paper and write all the if/else statement when you are debugging the application. Further, less code is written in the function.

## Takeaways

>"Failures, repeated failures, are finger posts on the road to achievement. One fails forward toward success." C.S. Lewis. 

To make the system more robust, the developer should throw any bugs that occur so that it is easy to catch and fix them immediately. A good way to deal with precondition and sanity check is to throw an exception or assertion. Once the condition doesn't satisfy, return it immediately. With failing fast and return early, we can reduce the cost of debugging and improve the quality of the system.

### Reference: 
- [martin-fowler-fail-fast](https://www.martinfowler.com/ieeeSoftware/failFast.pdf)
- [The Fail-Fast Principle in Software Development - DZone Agile](https://dzone.com/articles/fail-fast-principle-in-software-development)
- [DRY code vs. WET code \| Codementor](https://www.codementor.io/joshuaaroke/dry-code-vs-wet-code-89xjwv11w)

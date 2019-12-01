---
layout: post
title: Want To Write Quality Code? Focus On Your Unit Test When Writing It
date: 2019-12-01 14:33
summary: Benefit of doing unit test for improving your code quality
categories: unit-test tech software-development
tags: unit-test tech software-development
---
![Photo by Luca Bravo](https://images.unsplash.com/photo-1488590528505-98d2b5aba04b?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80)

Writing code with unit tests takes a long time to write your feature. It is more complicated. When you have a tight deadline, you are trying to make the feature works. You design the system that you think other engineers will easily maintain it. However, what will be the metric to create such a high-quality code and robust system?  How did you know, before you deploy your application to the world that it won't fail?

The answer to that, my friend, is a writing unit test while you write your features.

My manager told us one day on our daily stand-up that the upper management has a sudden request for our team to implement the super bundle, smart discount, features for the launch. It was October 22nd, and we have less than 1 month to create this service from scratch. The atmosphere of the meeting gets solemn, more like nervous. We don't know if we can finish implementing this service before the launch. However, it is critical. The stakeholder, investors, and consumers expect Disney to provide a super bundle feature upon launch. Therefore, not only that we have to implement this service, but we also tested the service at scale. 

Our team's philosophy has been test-driven. Therefore, unit test has always been the core of creating any of the projects. We strictly account for code coverage, even when the timeline is tight on the project. We are having this philosophy increase our code quality and create a robust system that helps smart discount service to successfully operates during the launch day.

When you treat every single line of your code writes a test case for it, you will start to think about how your system invoked. You will begin to think about how you need coverage for every line in your test suite to get the full coverage. You will write code more succinctly because more lines of code equal to more lines to cover in the test coverage. You will put more effort into refactoring your system before you push your feature for code review. Your team's code quality will also improve.

## Documentation
When you start writing unit tests on every code that you write, you create documentation of the system. New developers that join the team can look at your unit test and learn the functionality of your order. 

## Better Design Decisions
The unit test tends to change a lot and will break a lot if the design is not robust. If you have trouble unit testing your functions, it is because you have so much functionality inside your code that it breaks SRP (Single Responsibility Principle). If you think about that unit testing your system, you will start to write less convoluted if/else statement in your code and fail fast to decrease test coverage that you need to have. You will think about dependency injection and another de-coupled strategy to make your function and system as loosely coupled as possible. 

## Facilitates Refactoring and Simplifies Integration
The unit test facilitates changes and simplifies integration. If you need to change every single line of code when you change an element in your function, you might also think about how you can refactor your system because it is too coupled. 

A unit test verifies the accuracy of each unit. The units are integrated into an application via unit testing. Integration testing will be easier because the unit test has verified each group.

Some people argue that code quality doesn't improve code quality because testable code isn't automatically equal to better code. I agreed. However, a lot of good code quality comes from unit testing. For instance, having too much IO in function indicates that you need to separate or inject the IO to make the code testable. It also results in SRP. Realize that you need to make the same operations when you implement something in your test-suite shows that you need to extract that operation out to a function so that it doesn't break the DRY rule. A system with a unit test will be much robust. A team that enforces test-driven principles has a high chance of creating a more robust system than a team that doesn't.

## Conclusion
The goal of unit testing is to segregate each part of the program and test that the individual components are working correctly. To write quality code, you should enforce yourself to unit test every single code that you write. Designing the system with having to unit test in mind will create a more robust system because it forces you to think through your design and what it must to accomplish before you write the code. Testing each piece of code helps you define the responsibility of that code. Lastly, it simplifies integration and makes the testing process much faster in the long run. Although unit testing your system increases the amount of time it takes to develop a feature, it will increase your team's efficiency and code quality in the long run.

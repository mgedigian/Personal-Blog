---
layout: post
title: How to Maintain Clean Code in your projects
date: 2019-08-25 23:16
summary: Software development process is more than you think. Creating readable maintainable code is paramount to create great coding culture.
categories: software-best-practice tech software-development
tags: teamwork best-practice software-development
---

There are a lot of rules about software development process in tech companies, and each of them have was developed to solve a problem to create maintainable, robust system. When you develop a solo home project, you usually have two steps: write code and push code using some sort of version control like git. However, developing  features with other developers in a large code base will be hard to manage, and create technical debt. Technical debt is like a credit card debt, the longer you ignore it the worse it is down the road. It will decrease developer productivity, and decrease job satisfaction. Therefore, each company ***NEEDS*** to have rules of maintaining clean code when they onboard engineers. The process of clean code can be divided into these 5 major steps:

> Define Rules -> Write Code -> Use Static Code Checker -> Code Review -> Merge

## Defining Rules
When a new member joins, senior engineer or team lead needs to onboard the new members with a set a rules on how they write code. This means defining conventions on writing code. Each company has its own convention on developing application. For example, Google has its own style guide on writing <a class="blue" target="_blank" href="https://google.github.io/styleguide/javaguide.html">Java code</a>, Twitter has <a href="https://twitter.github.io/effectivescala/" class="blue" target="_blank">effective scala</a>, code convention for scala programming. If you have a team that cares about code quality, that is good beause you can cooperate with them to create or establish that rule. Otherwise, you will need to use your persuasion skills to persuade others, and business unit on why clean code is important. Team needs to create readable and maintainable code to decrease technical debt, so that business won't spend more resources and time to fix bigger problems in the future. In addition, you cannot be the only one who cares about code quality. Why? Because it is like swimming againts the current. You might advance forward a little bit, but you will feel furious and unproductive every time developing feature and eventually burn out. You and your team need to be at the same mindset on creating better code culture. It might take some time, effort and discipline, but the outcome will create faster development speed and job satisfaction.

Once, your team have defined rules and convention, you can start developing features and write code.

How to maintain each and everyone of your team abide on the define rules? If you have a team of 2, it is easy to persist writing code based on conventions. However, how can you be sure if those rules applied when you have more than 10 engineers in your team?

## Static Code Checker to the Rescue
Static code analysis tools is a method for computer program that is done by examining the code without compiling them. It is the best way to check if your code smells. Example of static code checker tools include: <a class="blue" href="https://eslint.org/" target="_blank">Eslint</a>, <a class="blue" href="https://www.sonarlint.org/" target="_blank">Sonarlint</a> ... etc. Each team or company will can setup convention rules. One of example of a popular coding convention for javaScript is by engineers in <a href="https://www.npmjs.com/package/eslint-config-airbnb" class="blue" target="_blank"> Airbnb</a>. It gives an easy way to notify developers to follow conventions and best practice while they are coding.

What should you do if you are assigned to a legacy code and need to build features on top of it? Many of the engineers don't have the priviledge to developed a solid new project from scratch. Looking at legacy code can be tough because it might not be readable. You can use the Boy Scout Rule, by Robert C. Martin:

<blockquote>
   <p> "Leave things BETTER than your found them." </p>
   <footer><cite title="Robert C. Martin">Robert C. Martin</cite></footer>
</blockquote>

It includes:
<mark>found dirty code -> need to complete task -> refactor</mark>

Sometimes the code that you need to refactor might be more than the time that you spent developing the feature. Therefore, you need to estimate if the effort is big or small. If it is small, refactor. If it is big, mark comment `TODO` on the code and set it with an issue tracker. 

Using this rule, small changes will add up, and will create better readable code for future maintainers for bug fixes, or add new features.

## Code Review
There are a lot of ways to do code review, and each company has different rules in how they conduct code review. This process is the general process that I see majority of the company have on code review.

The importance and benefit of code review, to name a few, can increase code quality, create positive effect on the team culture, and recognition on code expertise through peers is a source of pride for many programmers.

At least 2 person doing code review. However, 1 person is also fine, but he/she should understand the underlying business logic and task that you are trying to complete. You are able to merge your code once all highlighted issue is addressed and discuss throughout. Further, if your code receive a comment, you should address all of them. Saying, *"Yes, I have fix based on your suggestions."* Or *"no that is not how it should work, because of x, y, z reason."*

Code review should also be tested with build tools, such as unit, and integration test to make the process faster and seamless. If you don't have a build tools, you can do pair programming on the screen. This way, reviewee are able to get instant feedback on the review. Every time when you want someone to conduct your code say, *“Hey, can you come over and take a look at my code, and see if there is anything weird on this?”*

Lastly, code review are ***classless***, being the most senior person in the team does not imply that your code doesn’t need review. Be respectful to the reviewee. While adversarial thinking is handy, it is not your feature and you can’t make all the decisions. If you can’t agree with your reviewee with the code as is, switch to real time discussion or seek for third person opinion.

## Conclusion
By stating rules when onboarding engineers, they are able to understand any expectations while they are developing new features. Create a static code checker, and an automated build tools can maintain readable code intact. Lastly, having a good code review process can not only increase code quality, but also create great culture to engineers in the team. ***Clean code is not “nice to have”. It is a necessity.*** Even when you are cranking a new feature for business requirement and need to push the code as soon as possible, avoid skipping through these development process as possible. Eternal vigilance is the price of all quality, and aiming for that habit can help you achieve it.



---
layout: post
title: What is really so special about JavaScript Closure?
date: 2020-02-13 06:50
summary: It is easy to explain what it is but hard to understand why you need to use them.
categories: tech JavaScript closure functional-programming programming
tags: tech JavaScript closure functional-programming programming
---

<img src="{{site.baseurl}}/images/what-is-really-so-special-about-javascript-closure-/Closure JS.png" alt="JavaScript Closure">


"Can you explain to me what a closure is?"

If you are a full-stack, frontend, backend, or do anything with JavaScript, you must hear of the term closure. If you are interviewing for a full-stack software engineer role, you might encounter a question asking you to explain what a closure is.

The questions stated above are easy to answer - just memorized the definition of Closure and gave a couple of examples.

The problem with this terminology is not about understanding the definition of ClosureClosure, but to understand why you might want to use it in your project.

Before we dive deep into how sturdy a closure is in JavaScript, let's understand what a closure is.

Closure is a function that has access to its 'outer' scope that it is defined. Therefore, it can access the value in the outer scope, even if the enclosing function is terminated.

In a scala, a closure is equivalent to currying.

Take this as an example:
```javascript
function takeOne() {
  let i = 0;
  return function incrementFunction() {
    return i++;
  }
}
```

The code above represents a function returning another function. However, after you evoke `takeOne` and get `incrementFunction`, `incrementFunction` can the local variable of `takeOne` even though `takeOne` has already terminated.


## Benefit of Closure

The first benefit of ClosureClosure is to preserve local variables within the scope. Since javaScript is a first-class function, developers often encounter name clashing, that will cause some unexpected output. Using ClosureClosure can help preserve the namespace in that scope, private variable. You can see this a lot in the past of jQuery code, where one defines a click method.
```javascript
$(function() {
  var selections = []
  $(".something").click(function() { // this closure has access to the outer variable selections
    selections.push("something") // this are able to get the outer function selections
  })
})
```

While this is indeed one of the use-cases of ClosureClosure, it might leave you thinking, "Is this really what the real purpose of closure?" You might still question the statement as to what is the general use case of ClosureClosure might be.

The second benefit, which is more of a general use case, is that it is useful in an asynchronous environment.

Imagine if you need to loop through a value in an array:
```javascript
for(var i = 0 ; i< 3; i++) {
  setTimeout(() => console.log(i), 3000)
}

```

What will the output of this program?

It prints `3` three times. Since, setTimeout is asynchronous, by the time the loop finishes, the outer scope `i` has also changed to 3, and the subsequent call to `setTimeout` during the loop triggers the callback and print `3`. 

How would you solve this problem?

There are many ways, including using an ES6 syntax `let` instead of `var` to define its scope at the block level and solve the issue. However, if they want you to solve this issue without using any ES6 feature, ClosureClosure is your answer.

```javascript
function printSomething(i) {
  setTimeout(() => console.log(i), 3000)
}

for(var i = 0; i<3; i++) {
  printSomething(i)
}
```

By just creating another outer function outside of `setTimeout`, you are defining a closure. The `i` value is preserved even after `printSomething` is terminated. The callback then prints `0 1 2` to the console.

That is the reason why ClosureClosure is powerful javaScript feature. You can use ClosureClosure to preserve the scope of the outer variable in an asynchronous environment.

## Another Example One
Let's imagine another example where you need to create a function that needs to call 3rd party API and aggregate the result and return it to the caller.

```javascript
function getAPI(cb) {
    setTimeout(() => cb("a"), 3000)
}

function getAPIB(cb) {
    setTimeout(() => cb("b"), 2000)
}

function getAPIC(cb) {
    setTimeout(() => cb("c"), 1000)
}

function aggregateValue() {
  var aggregateData = []
  
  // your implementation here
}
```

Before you continue reading, about the solution, pause for a second and think about how you can solve this without promises, async/await.

We can leverage the power of ClosureClosure to preserve the scope of the function and to stop `aggregateValue` to return early by using callbacks.

Since `getAPI`, `getAPIB`, `getAPIC` all uses a callback function, you can create a callback function that increments the number of count of API called so far. Once the number of API called so far counter exceed 2, call the return callback value.

```javascript
function aggregateValue(cb) {
  var aggregateData = []
  var numberAPICalledSoFar = 0
  
  function callback(value) {
    aggregateData = [...aggregateData, value]
    if(numberAPICalledSoFar < 2) {
      numberAPICalledSoFar++;
    }else {
      cb(aggregateData)
    }
  }
  getAPI(callback)
  getAPIB(callback)
  getAPIC(callback)
}
```

The above code leverages, again, the power of ClosureClosure, to preserve the local variable of the enclosing function when it is triggered. As `getAPI` finishes its called and evoke the callback function, the callback function access the outer scope `aggregateValue` to increment the number of counts that the API finishes executing. The `aggregateData` then return with a callback from the outer `aggregateValue` function that needs all the aggregate data from all the 3rd party API.

Run this function:
```javascript
aggregateValue((ans) => ans.foreach(console.log))
```

## Takeaway
- Although ClosureClosure is taught in the first class of javaScript language, ClosureClosure is a more "advanced" feature of javaScript.
- The problem that is hard about ClosureClosure is not its concept, but its benefit, and why you, as a JavaScript developer, needs to understand its use case.
- The general use cases of ClosureClosure lies within solving computation in an asynchronous environment.

All the source code in this tutorial is [here](https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ClosureTutorial).

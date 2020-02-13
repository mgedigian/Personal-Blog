---
layout: post
title: Cracking the Combination Recursive Riddle
date: 2020-02-13 07:00
summary: To Choose or not to choose
categories: functional-programming programming scala algorithm jobs
tags: functional-programming programming scala algorithm jobs
---
![photo-1556743868-0c1460d5bc8e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1267&q=80](https://images.unsplash.com/photo-1556743868-0c1460d5bc8e?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1267&q=80)

During the late 19th century, mathematicians used the principles defining a function by induction. Dedekind, Peano 1889, uses induction to define and prove his <a href="http://www.people.cs.uchicago.edu/~soare/History/compute.pdf" target="_blank">fifth axioms</a> for positive integers using induction. Later, his fifth principle of a positive integer has been called primitive recursion. From then on, the concept of recursion has played an essential role in the foundation of mathematics.

Proof by induction is a way to prove a concept, theory, or algorithm that it works by justifying every step of that computation. It goes like this - you can prove an arbitrary statement n by first proving that the statement is true when n is 1 and then assuming that it is also true when n = k, and showing it is valid for n = k+1.

Recursion was derived from the concept of induction. A recursive function works by breaking down a problem into smaller parts by calling itself. Then, there is the base case, which is when n = 1. 

Therefore, a lot of algorithm logic can be derived from breaking the more significant problem statement to a smaller problem. And the combination is one of them.

When I learn Scala programming language and functional programming, I realized how every single call the function is recursive. You want to loop through a list and get the last two elements of the List, you have to use pattern matching and keep calling yourself until it hits the base case which is the last element of the List and returns that last element to the previous call stack.

A combination problem is a classic recursion problem that we often encounter in a software engineer interview questions. It is the problem when the interviewee stops his/her breath, scratch his/her head on how to start tackling the problem. Therefore, it is the right question for an interviewer to go through their interviewee's brain and see how they tackle down a complex problem.

Let's do an example - Create a combination function that returns all the possible combinations of a List of Integer.

```scala
val lst = List(1,2,3)
```

If we break down the list to only 1 elements ` List (1)`, what are the possible combination it can have?

### To Choose or not to choose
The answer is 2. Why? Because you either can pick that one element, `1`, or you don't.

From this observation, we can now start to figure out how that concept can translate into code.

```scala
def combination(len:Int, i:Int lst:List[Int]):List[List[Int]] = {
  // here you either choose the value or you didn't
  val includedList = lst.head :: combination(len, i+1, lst.tail) // choose the current head (this will not work, because you need to wrap the head inside the List(head :: Nil))
  val notIncludedList = combination(len, i+1, lst.tail) // skip the current head
}
```

Noticed at the above code, no matter what happened, the index `i` value will always increment, moving the pointer to the next element in the List. At one recursive call, we include `lst.head` , which is the chosen value. On the other hand, we do not include the `lst.head`, skipping this current value.

Every recursive function needs to have a base case. When I think about the base case, the first thing that comes to mind is what happened when the List is empty? What kind of value should I return?

In this case, if the list is empty, the function should return an empty list because we have nothing to choose.

The function becomes something like this:
```scala
def combination(len:Int, i:Int, lst:List[Int]):List[List[Int]] = {
  if(len == i) {
    List(Nil)
  }
  else {
      val includedList = combination(len, i+1, lst.tail).map(lst.head :: _) // choose the current head
      val notIncludedList = combination(len, i+1, lst.tail) // skip the current head
      includedList ::: notIncludedList
  }
}
```

The code above technically is good to go, because it produces the right output. However, in Scala, we can omit the `i` value, and traverse down the list by traverse the `tail` of the List. Let's refactor the above working solution more concisely.

```scala
def combination(lst:List[Int]): List[List[Int]] = {
  if(lst == Nil) {
    List(Nil)
  } else {
    val includedList = combination(lst.tail).map(lst.head :: _)
    val notIncludedList = combination(lst.tail)
    includedList ::: notIncludedList
  }
}
```

And combine it with pattern matching:
```scala
def combination(lst:List[Int]) : List[List[Int]] = lst match {
  case Nil => List(Nil)
  case h:rest => combination(rest).map(h :: _) ::: combination(rest)
}
```

This will be the simplest form of finding all the subset in a List. Therefore, `List(1,2,3)` will have `List(List(1), List(2), List(3), List(1,2), List(1,3), List(2,3), List(1,2,3), List())`.

That's it! That is the basic structure of combination. Most of the other combination problems involved some constraints in computing the algorithm further. Most of the dynamic programming algorithms can derive from the combination algorithm. 

For instance, the prevalent <a href="https://en.wikipedia.org/wiki/Knapsack_problem" target="_blank">knapsack problem</a>, you either choose this current value to include in a collection if the bag is not full or skip it. Once you find all the combinations, you compute the most value out of the combination that you choose. We can further derive all the pattern by creating an iterative approach, or memoized the computation with `LazyList`.


## Takeaway:
- A recursive algorithm is derived from proof by induction.
- All iterative computation solutions can be derived from the recursive function.
- The two main activities of the combination are to choose or not to choose.


## Food for thought
- At the observation stage, `lst.head` is prepended with the rest of the combination. However, at the end of the solution, I used the `map` function to prepend the head in the result. Is there any difference?

All the source code in this tutorial is [here](https://github.com/edwardGunawan/Blog-Tutorial/tree/master/ScalaTutorial/combinationTutorial).



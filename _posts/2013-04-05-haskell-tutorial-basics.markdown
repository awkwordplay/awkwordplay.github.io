---
layout: post
title:  "Haskell tutorial - Basics"
date:   2013-04-05
categories: programming
tags: functional-programming, haskell, tutorial haskell-tutorial
image: /assets/article_images/2014-08-29-welcome-to-jekyll/desktop.jpg
---

<h3>Expressions vs. statements</h3>

<p>
    If you are entirely new to programming please visit this
    <a href="http://en.wikipedia.org/wiki/Expression_(computer_science)">Wikipedia entry about expressions</a>.
    The first line contains a couple of terms wich will be our main vocabulary in this post.
</p>

<pre>
3       -- Constant
"Hello" -- Constant
x       -- Variable
y       -- Variable
1 + 4   -- Expression, when evaluated, yields a value (5)
</pre>

<p>
    Haskell is an expression based language. There are no statements in it, like in imperative ones. The following
    example illustrates the difference between an expression and a statement.
</p>

<pre>
// JavaScript
var condition = true
var x
if (condition) {
    x = 3
} else {
    x = 5
}

-- Haskell
let condition = True
    x = if condition
        then 3
        else 5
</pre>

<p>
    As you can see, in Haskell the if expression returns a value, while in JavaScript the if is a statement, and you modify the value of x
    from a branch of the if. Now, modifying values is something you do rarely in Haskell, because it is a "pure" language.
    Once you assigned a variable a value, you can never change it. In this sense, these variables are very close to their cousins in math.
    It is worthy to note that branches of an if expression must return values which has the same type. Thus, the following example causes
    a type error:
</p>

<pre>
> if True then 1 else "Hello"

<interactive>:2:14:
    No instance for (Num [Char])
    arising from the literal `1'
    Possible fix: add an instance declaration for (Num [Char])
    In the expression: 1
    In the expression: if True then 1 else "Hello"
    In an equation for `it': it = if True then 1 else "Hello"
</pre>

<p>
    The error itself may look scary for a Haskell beginner, but fear not, you will develop an intuition for them pretty soon.
    Please note that I use the ">" character to indicate the prompt in GHCi.
</p>

<h3>Function application</h3>

<p>
    Function application may be quite unusual for programmers coming from imperative languages: there are no parentheses, only spaces:
</p>

<pre>
// JavaScript
add(1, 2)

-- Haskell
add 1 2
</pre>

<p>
    This allows us to spare a couple of parens. You can evaluate an expression before passing it to a function too:
</p>

<pre>
// JavaScript
add(1, sub(4, 3))

-- Haskell
add 1 (sub 4 3)
</pre>

<p>
    In Haskell, expressions are evaluated only when needed, which is called non-strict, or lazy evaluation. The following snippet does not actually
    raise an error, contrary to the intuition.
</p>

<pre>
let list = [1, 2, error "Hey.", 3]
</pre>

<p>
    In a strict language, we could not even store the list, because the evaluation of the error would happen before constructing the list.
    In Haskell, this is not the case. The error will only be raised if that value will be used. This requires an entirely different mindset to predict
    the performance characteristics of a program, but at the same time it allows us to glue our existing functions in a more modular fashion.
</p>

<p>
    One of the most enlightening example of the modularity provided by lazy evaluation is the following:
    imagine that we need to get the smallest element in a list and we have no minimum function, but
    we have a function which can sort us the list. Now, if we sort the list in ascending order, the first element will be the smallest, but sorting an entire list
    to only get the first element is entirely wasteful. It is, in strict languages. But in non-strict ones, the sorting will only happen if needed: if we
    only need the first value, the sorting algorithm will only do the sorting until it can produce that one value we need: it will do no unnecessary work!
    If that did not blow your mind I suggest you to stand in the corner until you realize how awesome it is.
</p>

<h3>Defining values</h3>

<p>
    Defining values at top level is easy:
</p>

<pre>
x = 3
y = "Hello"
z = y
</pre>

<p>
    Ignoring the precise type of those expression, I think you can guess that x will be a number, y is a string,
    and z has the same value and type as z.
</p>

<h3>Function definition</h3>

<p>
    This is the exciting part! We will do actual computation here. The fact is, Haskell is an shamelessly concise language:
</p>

<pre>
add x y = x + y
</pre>

<p>
    That's it! That line will add two numbers. Admittedly not too useful, but hey! We have to start somewhere. Using it is also as easy as ABC:
</p>

<pre>
> add 3 5
8
</pre>

<p>
    To venture deeper into the language we will have to befriend the types, but that is a story for an other day.
</p>
---
layout: post
title:  "Haskell tutorial - Thinking recursively"
date:   2013-04-10
---

<p>
    To understand this post, first you should learn...
</p>

<h3>Patterns and pattern matching</h3>

<p>
    Patterns allow you to deconstruct certain data structures with the same operators what you use to construct them.
    This only works in certain places, but we focus on function definitions now.
    Examine the following one:
</p>

{% highlight haskell %}
summa :: [Int] -> Int
summa list = if length list == 0
    then 0
    else head list + (summa (tail list))
{% endhighlight %}

<p>
    This contains no pattern matching yet. We have to do a lot of grunt work by hand, checking the lenght of the list,
    accessing the first element (head), calling the function recursively on the remaining list. Pattern matching can help to
    lessen the pain:
</p>

{% highlight haskell %}
summa :: [Int] -> Int
summa [] = 0
summa (x:xs) = x + summa xs
{% endhighlight %}

<p>
    Much nicer, right? The cons operator (:) appends an element to a list, so the above quite intiutively reads like this: if we have an empty
    list, the result is 0. If we have a list with at least one element (x appended to the possibly empty xs), the result is the first element +
    summa the remaining list.
</p>

<p>
    Another good example is the classic (but very inefficient) definition of Fibonacci numbers: 
</p>

{% highlight haskell %}
fib :: Int -> Int
fib 0 = 0
fib 1 = 1
fib n = fib (n-1) + fib (n-2)
{% endhighlight %}

<p>
    The above calculates the fibonacci number in the nth position of the Fibonacci sequence.
</p>

<h3>Thinking recursively</h3>

<p>
    Every iterative algorithm and can be expressed as recursion and <a href="http://en.wikipedia.org/wiki/Vice_Versa">vice versa</a>.
    In Haskell, recursion is the norm, since iteration is impossible to do without mutable variables. This can be tricky to imagine for newbies,
    so let's look at a couple of examples.
</p>

{% highlight haskell %}
-- Counts how many element satisfies the predicate in a list.
count :: (a -> Bool) -> [a] -> Int
count p xs = f 0 xs
    where
        f c [] = 0
        f c (y:ys) =
            let v = if p y then 1 else 0
            in f (c + v) ys
{% endhighlight %}

<p>
    The where construct in the above example is a simple way to define local functions and variables.
    Where is similar to the <b>let ... in</b> form, the only difference is that ith let the definitions come before the expression body, with where
    they stand after it.
</p>

<p>
    Our count function works by defining a recursive local function called <b>f</b>, which passes state as one of its arguments.
    Every recursive invocation can be tought of as one iteration. The same function in JavaScript would look like this:
</p>

{% highlight haskell %}
function count(pred, list) {
    var counter = 0
    for (var i in list) {
        if (pred(list[i]) == true) {
            counter++
        }
    }
    return counter
}
{% endhighlight %}

<p>
    A side note: a nice thing in Haskell is that we can be lazy (pun intended) and use the existing functions to express our thoughts. It really feels like synthesis.
    Instead of controlling every nuance of the code, we can usually achieve our goals in a couple of steps. Our count function would look like:
</p>

{% highlight haskell %}
count :: count :: (a -> Bool) -> [a] -> Int
count p xs = length $ filter p xs
{% endhighlight %}

<p>
    But currently we care about writing these functions ourselves to learn the thought process behind them. However, it would be silly to ignore the wide range of
    already written functions in the Haskell standard library (Prelude). Yes, you've heard it! Unlike in C++ or a lot of other langauges, the Haskell Prelude
    is full of readable, idiomatic functions. Reading them is a pleasure. Let's look at a couple of them.
</p>

{% highlight haskell %}
map :: (a -> b) -> [a] -> [b]
map _ []     = []
map f (x:xs) = f x : map f xs
{% endhighlight %}

<p>
    The implementation is map is actually very straightforward, we apply the function f to the first element of a list, and append that element to the 
    remaining of the also mapped list. If you are not already blown away by the elegance of it, I suggest you to learn Java.
</p>

{% highlight haskell %}
filter :: (a -> Bool) -> [a] -> [a]
filter _pred []    = []
filter pred (x:xs)
  | pred x         = x : filter pred xs
  | otherwise      = filter pred xs
{% endhighlight %}

<p>
    Filter is quite similar to map, but there is new syntax here: guards. Guards are basically pretty switches, the first case evaluating to true
    will be evaluated. The definition reads: if the list is empty, return the empty list. If the list is not empty, check if the predicate holds true on
    the first element of the list, if it does, append the given element to the remaining of the list.
</p>

<p>
    Most people when hear that Haskell has no mutable variables (not counting the IO monad, more on this later) are terrified by the idea
    and they think that working with immutability is half impossible and half crazy. But in fact, recursive solutions are often very concise
    and easy to write. Pattern matching helps us to visualize corner cases (empty list, zero, etc) when we have to terminate our recursion, or act differently.
</p>

<h3>Performance</h3>

<p>
    Haskell lends itself especially well to recursive expressions. In imperative languages, recursion is considered slow, and iteration is preferred.
    Since Haskell is optimized for recursion, you don't have to fear a performance
    penalty. <a href="http://en.wikipedia.org/wiki/Tail_call">Tail call optimization</a> actually ensures that when a function calls itself in tail position
    (see the summa example), the resulting machine code will be identical to loops.
</p>
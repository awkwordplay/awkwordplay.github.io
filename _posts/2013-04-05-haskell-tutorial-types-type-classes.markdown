---
layout: post
title:  "Haskell - Types, type classes"
date:   2013-04-05
---

<p>
    Haskell has an incredibly rich type system which puts to shame almost all other langauges with the possible exception of
    <a href="http://en.wikipedia.org/wiki/Category:Dependently_typed_languages">dependently typed languages</a>, which are under heavy
    development to become useful languages suitable for practical work (oh, not this practical story again!*).
</p>

<p>
    *For the not yet insiders, Haskell is considered impractical by a big percentage of programmers, and that is just simply not true.
</p>

<h3>Reading type signatures</h3>

<p>
    Please allow me to go a little slower with this post so you can develop an intuition for the types of Haskell. If you know generics from
    Java, or C++ I'm sure you know how much of a hassle is to write generic code. And even worse, some languages doesn't even have this feature
    (I'm looking at you, <a href="http://golang.org/">Go</a>!). Haskell gives you generics for free. There is no distinction in effort between generic and
    non-generic code.
</p>

<p>
    Let's look at a type signature:
</p>

{% highlight haskell %}
> :t 
"Hello" :: [Char]
{% endhighlight %}

<p>
    [Char] means a "list of Chars". By the way, all concrete type names in Haskell start with uppercase, while variables, generic types and
    functions start with lowercase. You can read :: as "has type", or "here comes the type of the previous thing" if you feel funkier. As you can see, Char is a
    concrete type, but what is an example of a generic type? There is a function called head, which gives you the first element of a list. Let's see the type signature
    of it:
</p>

{% highlight haskell %}
> :t
head :: [a] -> a
{% endhighlight %}

<p>
    The -> symbol separates the types of arguments and the return type. The last one is always the return type (every function has one), and all other are arguments.
    So a function having the signature "String -> Int -> Bool" will need a String and an Int as argument, and returns a Bool. So what does the signature of head tells
    us? It says it has only one input argument, a list containing values having the type "a", and it returns a value having the type "a". But what the heck is "a"?
    Where is that defined? What does it mean, you may ask. The answer is pretty simple, it means nothing. It is a placeholder, for any type, thats why it is generic.
    Of course, we may call it b, or c, or e, or anything, but there is a certain beauty in following the order of the alphabet.
</p>

<p>
    Please note that both a in that signature denotes the same type! If we have an "a" and a "b" in a type signature, they may denote the same type, or their
    types may differ, but the same letters MUST denote the same type. With this knowledge, let's see a couple of type signatures and function names and meditate
    on what may they do.
</p>

{% highlight haskell %}
> :t id
id :: a -> a
{% endhighlight %}

<p>
    The <a href="http://en.wikipedia.org/wiki/Identity_function">name of the function</a> gives it away, but let's think about it a bit. The id, or identity
    function takes a value of any type, and returns a value of the same type. Now, what may possibly happen inside that function? Could it double or triple
    its input? Definitely know, because we don't know if it is a number! Can we compare it to True or False? No, because we don't know if it is a Bool! We don't
    know anything about that type. We don't know what functions are compatible with it.
</p>

<h3>Type classes</h3>

<p>
    And with this we arrived to type classes. A type class is just a thing which tells us what can you do with a generic type like that.
</p>

{% highlight haskell %}
> :t (+)
(+) :: Num a => a -> a -> a
{% endhighlight %}

<p>
    Whoa! The part preceding the <b>=></b> tells us which type class <b>a</b> must belong to. To be able to add together values, the type of those values must be an instance
    of the type class Num. Yes, you have read it right! As values are instances of a type, types are instances of a type class. If you know interfaces from Java or Go,
    type classes are a very similar concept to them, they are a way to define a desirable trait of a type, so you don't care about the concrete type, as long as they
    have those give properties. What is the Num type class, for example?
</p>

{% highlight haskell %}
> :i Num
class Num a where
  (+) :: a -> a -> a
  (*) :: a -> a -> a
  (-) :: a -> a -> a
  negate :: a -> a
  abs :: a -> a
  signum :: a -> a
  fromInteger :: Integer -> a
        -- Defined in `GHC.Num'
instance Num Integer -- Defined in `GHC.Num'
instance Num Int -- Defined in `GHC.Num'
instance Num Float -- Defined in `GHC.Float'
instance Num Double -- Defined in `GHC.Float'
{% endhighlight %}

<p>
    There you can see a definition of a type class. The Num type class has the following functions defined: +, *, -, negate, abs, signum and fromInteger.
    The <b>:i</b> command is so helpful it even lists which types are the instances of this type class: Integer, Int, Float and Double.
    We can play around and see if they indeed work with the given functions.
</p>

{% highlight haskell %}
> 1.2 + 3.2
4.4
> 5 + 10
15
> negate 4.5
-4.5
> abs (-20)
20
{% endhighlight %}

<p>
    If you try to query the type of a number with a decimal point in it you may notice that it is not a Num:
</p>

{% highlight haskell %}
> :t 1.2
1.2 :: Fractional a => a
{% endhighlight %}

<p>
    Oh-oh! The type of 1.2 is an instance of the type class Fractional, how come we used it with functions which need types implementing the Num type class
    then? Let's investigate...
</p>

{% highlight haskell %}
> :i Fractional
class Num a => Fractional a where
  (/) :: a -> a -> a
  recip :: a -> a
  fromRational :: Rational -> a
        -- Defined in `GHC.Real'
instance Fractional Float -- Defined in `GHC.Float'
instance Fractional Double -- Defined in `GHC.Float'
{% endhighlight %}

<p>
    The most interesting part here is that instead of having a simple type class declaration like "class Fractional a where", we have
    "class <b>Num a</b> => Fractional a where". This means that the given type which implements the Fractional type class must also implement
    the Num type class. Every Fractional is a Num but of course not every Num is a Fractional. That's why we can use Fractionals in place of Nums.
</p>
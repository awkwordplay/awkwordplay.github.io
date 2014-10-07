---
layout: post
title:  "Haskell tutorial - Lists"
date:   2013-04-06
---

<p>
    In the previous post we met most of the basic types:
</p>

<pre>
Name        Example     Description
--          -------     -----------
Int         300         Machine word sized integer.
Integer     300         Infinite sized integer.
Float       3.2         Floating point numbers.
Double      3.2         Double precision floating point numbers.
Char        'a'         A single character.
Bool        True        Boolean type, can take two values: True or False.
</pre>

<p>
    As you can see, there is nothing complex about them. Lists are a bit different. The term "list" is not a concrete type. We can talk about
    a list of Chars, a list of Ints, a list of Floats and so on. Please meditate on this since this is an incredibly important and powerful concept: some
    abstract types need one or more additional types to produce a concrete type. And lists are a very good example to show that. First, note that we could mentally
    replace the type <b>[a]</b> with <b>List a</b>, as the [] is just syntactic sugar. In this regard, the list type is special, but only in cosmetics.
</p>

<h3>Higher kinded types</h3>

<p>
    If you want to amuse girls, you may want to learn the proper terminology for those above mentioned "abstract types": they are called higher kinded types.
    You can also refer to <b>List</b> as being a type constructor, while <b>List a</b> is a lower kinded type, or in layman terms a concrete type, or simply a type.
    GHCi is still your friend, there is a command, :k, which will return you the kind of a type.
</p>

{% highlight haskell %}
> :k Int
Int :: *

> :k []
[] :: * -> *

>:k [Int]
[Int] :: *

> import Data.Map
> :k Map
Map :: * -> * -> *
{% endhighlight %}

<p>
    Wohoo! We are advancing so fast. The <b>-></b> symbol means the same as in function type signatures: it separates the inputs and the
    output of a function. Type constructors are functions too, the only difference is that they work on types, and not on values. Let's interpret those lines.
    Int is a lower kinded type, [] is a type constructor, [Int] is again a simple lower kinded type,
    and we imported the module Data.Map just for the sake of it. It serves as an example about how a higher kinded type can need more than one type to produce
    a concrete type. List needs one, why? The elements in that list have a type too. But a Map can needs one type for the key, and one for the value.
</p>

<h3>Creating lists</h3>

<p>
    To create a list, we have to list values belonging to the same type, separated by commas, inside square brackets:
</p>

{% highlight haskell %}
> let x = [1, 2, 3, 4, 5]
> :t x
x :: [Integer]
> let y = [True, False, False]
> :t y
y :: [Bool]
{% endhighlight %}

<p>
    We can do enumeration very easily too (if the given type is an instance of the Enum type class):
</p>

{% highlight haskell %}
> [0..20]
[0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20]
> ['a'..'z']
"abcdefghijklmnopqrstuvwxyz"
{% endhighlight %}

<h3>Lazy evaluation</h3>

<p>
    Once thing you must keep in mind that since Haskell has non-strict evaluation, you can work with very large, even infinite lists without killing your machine.
    We already now the head function.
</p>

{% highlight haskell %}
> -- This is how you define an infinite list:
> let x = [1..]
> head x
1
{% endhighlight %}

<p>
    In strict languages the infinite list would halt our program (at least this thread), but not in Haskell. When we assign x the infinite list, the list is not
    constructed immediately, only when needed, and even then, not the whole list, but only the needed parts will be calculated. This may seem contrived at first,
    I mean what was the last time you needed infinitely large lists, right? Our RAM is not infinite?  When programming in Haskell,
    we usually word our thoughts in a way which is very close to mathematics. That's why the language is sometimes called an "executable specification language".
    The problem is that math definitions do not match the underlying hardware, and this creates performance problems. Non-strict, or lazy evaluation helps to lessen
    the severity of this problem by allowing us to word our algorithm in a concise and elegant way, without going into details and micro-manage every variable,
    without suffering a performance penalty.
</p>

<h3>Working with lists</h3>

<p>
    The module <a href="http://hackage.haskell.org/packages/archive/base/4.3.0.0/doc/html/Prelude.html">Prelude</a> is imported by default. It contains
    very useful list manipulation functions. It is worthy to know most of them since many problems can be solved with list manipulation.
    The most used functions are:
</p>

<h4>length</h4>

{% highlight haskell %}
> :t length
length :: [a] -> Int
> length [0..10]
11
{% endhighlight %}

<p>
    Length (amazingly) returns the length of the list.
</p>

<h4>(:)</h4>

<p>
    The cons operator makes it possible to append an element to the beginning of a list. This is an O(1) operation, unlike appending to the end of a singly
    linked list.
</p>

{% highlight haskell %}
> :t (:)
(:) :: a -> [a] -> [a]
> 1:[1,2,3]
[1,1,2,3]
{% endhighlight %}

<p>
    This operator is right associative, we can conveniently append mutiple elements:
</p>

{% highlight haskell %}
> :i (:)
data [] a = ... | a : [a]       -- Defined in `GHC.Types'
infixr 5 :
> 1:2:3:[]
[1,2,3]
{% endhighlight %}

<h4>(++)</h4>

<p>
    The ++ operator concatenates to lists.
</p>

{% highlight haskell %}
> :t (++)
(++) :: [a] -> [a] -> [a]
> [1,2,3] ++ [10,20,30]
[1,2,3,10,20,30]
{% endhighlight %}

<h4>head, last, init, tail, (!!)</h4>

<p>
    These functions allow you to extract elements and sublists from lists easily.
</p>

{% highlight haskell %}
> head [1..5]
1
> last [1..5]
5
> init [1..5]
[1,2,3,4]
> last [1..5]
[2,3,4,5]
> [1..5]!!2
3
{% endhighlight %}

<p>
    Lists are zero indexed in Haskell, which means we refer to the first element as not the first, but rather as the 0th element.
</p>

<h4>map</h4>

{% highlight haskell %}
> :t map
map :: (a -> b) -> [a] -> [b]
{% endhighlight %}

<p>
    Map is a higher order function, which means that it either takes functions as its input or returns a function. The () in the signature means a function.
    Map takes a function and a list and it builds a new list by applying that function to every element of the input list. Let's say we have a function double, which
    can double a number, and a list of numbers. Issuing <b>map double list</b> will return a list with the doubled numbers:
</p>

{% highlight haskell %}
> let double x = x * 2
> let myList = [1..10]
> myList
[1,2,3,4,5,6,7,8,9,10]
> map double myList
[2,4,6,8,10,12,14,16,18,20]
{% endhighlight %}

<h4>filter</h4>

{% highlight haskell %}
> :t filter
filter :: (a -> Bool) -> [a] -> [a]
{% endhighlight %}

<p>
    Filter takes a predicate (a function returning a boolean value) and a list, and returns a new list containing only the values which satisfies the predicate.
    An example would be to filter numbers based on whether they are even:
</p>

{% highlight haskell %}
> :t rem
rem :: Integral a => a -> a -> a
-- Integral is a type class implemented by both Int and Integer types.
-- Do :i Integral for more information

> let isEven x = rem x 2 == 0
> filter isEven [0..20]
[0,2,4,6,8,10,12,14,16,18,20]
{% endhighlight %}

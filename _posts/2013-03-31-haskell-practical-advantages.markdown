---
layout: post
title:  "Haskell - Practical Advantages"
date:   2013-03-31
---

<p>
    There are more than a couple of meatless posts on the net about how the magical (not at all) functional programming paradigm is entirely different (no it isn't)
    from everything you are used to, and how will it save the world (no it won't).
</p>

<p>
    So let's try to approach Haskell, which is the <a href="http://stackoverflow.com/a/809983/441291">800-pound gorilla</a> in FP (functional programming) land, from a practical
    angle, without empty buzzwords. If you think that this article also lacks meat, well, you are wrong because I am right, but nevertheless slap me in the face.
</p>

<p>
    I will intentionally remain in shallow waters to be easily digestible by the broadest possible audience.
</p>

<h3>Static type system with type inference</h3>

<p>
    Most programmers thinkink about static typing imagine something like this:
</p>

{% highlight java %}
    Foo* foo = new(Foo);
{% endhighlight %}

<p>
    Usually when coders argue about wether static or dynamic typing is teh sh*t, they contrast the dynamically typed languages with C++/Java school of static typing (see above).
    Sacrificing some sophistication the usual conclusion can be reduced to "static typing can catch more bugs but you have to type more" (let's not go into unit testing now).
    If that's what you tought too, let me introduce you the <a href="http://en.wikipedia.org/wiki/Hindley%E2%80%93Milner">Hindley-Milner type inference</a>. Basically with the help of it,
    the compiler can type check your code without requiring you to annotate your types in even one place (I lied, in rare cases, it may be required).
</p>

<p>
    And no, Google's new language, Go does not have type inference, at least nothing comparable to what you can find in Haskell.
    What the Go evangelists try to sell as type inference is a very little subset of (full) type inference. The Go way:
</p>

{% highlight go %}
func nonsense(a int, b string) int {    // <- You have to annotate type or compilation fails.
    c := 42 // <- "Initialization and declaration" which is being sold as type inference
    return a + c
}
{% endhighlight %}

<p>
    The Haskell way:
</p>

{% highlight haskell %}
nonsense a b = let c = 42 in a + c
{% endhighlight %}

<p>
    That Haskell snippet will compile. What will be the type of it? You can query it by typing ":t nonsense" in GHCi
</p>

{% highlight haskell %}
> :t nonsense
nonsense :: Num a => a -> t -> a
{% endhighlight %}

<p>
    That type signature basically means: nonsense is a function, with two arguments, first is a, which can be any type, as long as that given type can be used as
    a number (is an instance of type class Num. Don't confuse type classes with Javaish classes, they are more like interfaces). The second argument can be any type, without
    restriction. Why? Because we did not use it! The return value has the same type as the first argument. This part can be quite tricky to nonhaskellers, because return type
    polymorphism is very rare in mainstream languages.
</p>

<p>
    As you can see, what we have here is the best of both worlds: compile time type checking without the requirement of annotating types by hand. Of course, type annotation is
    useful for humans, but the Haskell compiler doesn't really care about it, he is clever enough to figure this out. In fact, a lot of times I (and I suspect others too, but I can't
    speak for them) I write the function without type annotations, ask GHCi for the type signature and paste that into the source file to help others and my 2 months older version of
    me understanding the code.
</p>

<h3>Generics without breaking a sweat</h3>

<p>
    As mindful readers may have already noticed in the above Haskell snippet, in Haskell you write generic code by default. You don't have to learn a template language
    (there is a thing called Template Haskell but that is another story), you don't have to buy a 24 inch monitor to write generic function signatures. That is the default and
    it comes for free.
</p>

<h3>Syntax without clutter</h3>

<p>
    One thing I noticed while learning Haskell, that the authors have a great sense of beauty. This can be generally said about the language as a whole, but it can be most easily
    seen when looking at the syntax. Here are a couple of constructs side by side with the Go version, as that is a language I used directly before Haskell thus I hopefully
    make no errors.
</p>

{% highlight go %}
// Switch case

// Go
a := 20
switch a {
    case 10:
            return "It's ten!"
    case 20:
            return "It's not ten :("
}
{% endhighlight %}

{% highlight haskell %}
// Haskell
let a = 20
in case a of
    10  -> "It's ten!"
    20  -> "It's not ten :("
{% endhighlight %}

{% highlight go %}
// Anonymous function

func (a, b int) int {
    a + b
}
{% endhighlight %}

{% highlight haskell %}
\a b -> a + b
{% endhighlight %}

{% highlight go %}
// Looping: transform a list/array

// Go
a := []int{1,2,3,4,5,6,7,8,9,10}
f := func(a int) int {
    return a+1
}
b := []int{}
for _, v := range a {
    b = append(b, v)
}
return v
{% endhighlight %}

{% highlight haskell %}
// Haskell
let a = [1..10]
    f = (+1)
map f a
{% endhighlight %}

<p>
    The differences really add up. There are some outrageous claims on the net about how much you can save with functional programming in terms of line count, but I think
    a good estimate is that you can at least halve the LOCs.
</p>

<h3>REPL</h3>

<p>
    Haskell is amongst the few statically typed languages what you can use interactively. GHCi is the Haskell compiler, and despite being a not too mainstream language, it is
    the Haskell platform is a breeze to use, even on Windows, its a one click install, and you can double click on any hs files and GHCi starts. GHCi is an incredible productivity
    boost. Just to name a few cool features: :t gives back the type of any expression, :i will display information about the given type, you can set GHCi to clock all expressions
    (display the milliseconds spent calculating the expression). The possibilities are endless, really.
</p>

<h3>Type signatures are telling</h3>

<p>
    The information dense type signature of Haskell functions allow us to effectively search for a function by approximate type with the help of a tool like
    <a href="http://www.haskell.org/hoogle/">Hoogle</a>.
</p>

<h3>Conclusion</h3>

<p>
    The general consensus on Haskell is that it's impractical. I think that opinion mainly stems from old experiences. I agree that it's still rough around some edges:
    I am particularly worried about the lack of stack traces. The thought of running it on production servers and not seeing the source of an exception frankly scares the
    sh*t out of me. But as a language it is very well designed, especially compared to the current mainstream languages.
</p>

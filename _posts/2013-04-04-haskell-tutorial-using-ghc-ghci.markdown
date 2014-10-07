---
layout: post
title:  "Haskell tutorial - Using GHC, GHCi"
date:   2013-03-31
---

<p>
    After installing the Haskell platform we have access to both the Glasgow Haskell Compiler (GHC) and the interpreter,
    GHCi. Throughout this tutorial series I will use GHCi, because it is interactive and easy to use. Nevertheless, here is
    how you compile a Haskell program with GHC:
</p>

<p>
    Let's say you create a file "C:/hsfiles/hello.hs". Copy and paste the following line into the file:
</p>

{% highlight haskell %}
main = print "Hello world."
{% endhighlight %}

<p>
    The main function is where your program starts. If you write a library, that will obviously have no main function, but now we want to
    produce a working executable, so here it is. Once you have the file with the above content, start the command line.
</p>

<pre>
> cd hsfiles
> ghc hello.hs
</pre>

<p>
    After this, you will have a file "C:/hsfiles/hello.exe". Success! If you double click on the exe you will only see a flash, because as soon as it prints
    the line "Hello world.", it quits. To avoid this, turn to the command line again and start the program from there. Hopefully you didn't close
    it, but if you did, step into the "C:/hsfiles" folder again.
</p>

<pre>
> hello.exe
"Hello world."
</pre>

<h3>GHCi</h3>

<p>
    Yay! You don't have to compile that hello.hs file though. If you double click on it, GHCi, the Haskell interpreter starts, and you can work
    with your file interactively. GHCi is a really powerful tool which enables you to work productively with Haskell. Once GHCi runs and the hello.hs
    file is loaded, you can call the defined functions (currently only main) by name. You don't even have to know the full function name, GHCi does
    autocompletion by default, for example: type m and tab, and it will print all functions starting with m.
</p>

<pre>
> main
"Hello world."
</pre>

<p>
    There are a couple of commands which makes your life easier. One of the most important is, especially while you are learning the ":t" command,
    which prints the type of a given value or expression.
</p>

{% highlight haskell %}
> :t main
main :: IO ()
{% endhighlight %}

<p>
    Don't be afraid if you don't understand that now, we will cover it later. The ":i" command is similar, but more detailed: it prints the
    place of definition, the definition itself in in case of records, or instances of a given type class in case of type classes.
    If you modify the source file and want to reload it, use the ":r" command.
</p>

<p>
    One more word on GHCi. When we work with the interpreter, we are already inside a thing called the "IO monad". That may sound scary, but all you have to
    remember now that when we define something with the help of the "let" construct, we do that because of that. The difference is:
</p>

{% highlight haskell %}
-- Define a function in a file
f x y = x + y

-- Define a function in the interpreter (GHCi)
let f x y = x + y
{% endhighlight %}
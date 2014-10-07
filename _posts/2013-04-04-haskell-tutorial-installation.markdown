---
layout: post
title:  "Haskell tutorial - Installation"
date:   2013-04-04
---

<p>
    This is the first post in my Haskell tutorial series. As I am currently teaching one of my friends Haskell, I thought documenting
    the process could benefit some. Often a different wording can shed light on an issue, so here is my take. The person in question knows
    a bit of <a href="http://golang.org/">Go</a>, but he is mostly a beginner to programming.
</p>

<p>    
    I will try to assume as little programming knowledge as possible but if you have never coded a line in your life you should consider
    consulting an other source, or picking an easier language to start with, depending on your goal. If you want to learn programming to
    create applications as soon as possible, I think Haskell may not be the best option because it requires a will of steel to get to a level
    where you can fluently express your thoughts and where you can read and use 3rd party libraries with ease. If your primary goal is
    to learn a beautifully designed language which implements a wide range of cutting edge features, but still very effective in the
    practical world with enough determination, then Haskell is unsurpassed at the time of writing this.
</p>

<p>
    Having experience with a language with closure support, and/or with a language having a static type system would be a plus. 
    I will try to provide parallel code examples in JavaScript too where it makes sense for those coming from imperative languages.
    JavaScript was chosen because it is the most popular language to date, and everyone has access to JS interpreters - just fire up a browser.
</p>

<h3>Get the Haskell Platform</h3>

<p>
    The easiest way to install Haskell is to download the <a href="http://www.haskell.org/platform/">Haskell platform</a>.
    It is a batteries included package, and it only takes a click to install. I use it on Windows which is generally a not too well supported
    platform by non-mainstream languages but the Haskell platform works flawlessly for me, and I don't hear complains about it in the Haskell IRC
    channel (#haskell on Freenode) either. That IRC channel by the way may is full of helpful people, so feel free to ask for help there,
    we are very welcoming to beginners. If you don't know how to use an IRC client, <a href="http://webchat.freenode.net/">here is an online one</a>.
    Don't forget the hash from the #haskell channel name when you try to log in.
</p>

<h3>Install packages with cabal</h3>

<p>
    After you installed the platform you can also install individual modules written by different Haskell programmers. There is a central repository
    where most packages can be found (those which are uploaded by their authors), it is called <a href="http://hackage.haskell.org/packages/hackage.html">
    Hackage</a>. If you find a package you would like to use you can issue the next command in the command line:
</p>

<pre>
cabal install pkg-name
</pre>

<p>
    Where pkg-name obviously changes depending on your needs. Cabal is the name of the tool which handles the module installation magic.
    This command will only work when the given module is in HackageDB already. If you downloaded a module yourself, to install, you must step into the
    directory where this module resides. You can recognise an installable module easily, search for files like:
</p>

<pre>
pkg-name.cabal
Setup.hs
</pre>

<p>
    When you are in the proper director, just issue <b>cabal install</b> without any modulename.
</p>

<h4>Finding modules</h4>

<p>
    There is no use in package manager tools if you don't know what do you want to install.
    There is a very useful for Haskell programmers <a href="http://www.haskell.org/hoogle/">Hoogle</a>, it lets you search for Haskell
    functions by name or signature. Type signatures in Haskell are very telling, I will explain later why.
</p>

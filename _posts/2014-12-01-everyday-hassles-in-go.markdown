---
layout: post
title:  "Everyday hassles in Go"
date:   2014-12-01
---

**This article is not baked yet**

Go became a reliable, if a bit simple minded friend of mine during the past years. I use it in my day job and for side projects alike. Despite using (and liking*) it a lot, I have to admit the language could be improved a lot by borrowing battletested ideas from more modern languages. Unfortunately, a lot of Go programmers are coming from untyped languages, which means they haven't yet acquired the taste for sufficiently expressive type systems. A snarky person might say, they suffer from the <a href="http://www.paulgraham.com/avg.html">blub paradox</a>.

This lack of perspective in the Go community further hinders the progress of the language - people do not exert enough force toward the authors (not like they seem to be crowd pleasers anyway) to better the language. While I am grateful for Go as a tool, I am slightly worried about it's potential educational effect - or the lack of it. Given it is backed by Google - the hype and exposure that brings - even design failures will be accepted as 'the way to do it' by a large number of people. People like the authors of Go has an immense responsibility when it comes to improving our industry as a whole.

To show the limitations of some of the archaic concepts present in Go - here are the analysis if some of the features (or the lack of them) I consider unfortunate, with use cases and accompanying code. Some of these will be highly subjective, and the examples may be quiet arbitrary. Most of the difficulties I encounter could be fixed by a relatively small number of changes (<a href="http://en.wikipedia.org/wiki/Pareto_principle">the 80/20 rule?</a>).

*Mostly due to the amazingly comprehensive standard library. For such a young language anyway. The documentation is also brilliant.

### Lack of generics

Perhaps the most elusive, but rather destructive aspect of the lack of generics in Go is how the language forces the user to go into uninteresting details while expressing ideas, distrupting the programmer's flow. Unfortunately a lot of Go programmers are coming from untyped languages, which means they haven't yet acquired the taste for sufficiently expressive type systems. A snarky person might say, they suffer from the <a href="http://www.paulgraham.com/avg.html">blub paradox</a>. For them, here are a couple of examples why generics make sense.

Here are some examples to demonstrate the effect:

#### Deduping slice elements

Deduping elements of a slice happens the following way in go:

{% highlight go %}
package main

import "fmt"

func main() {
	// Given the following list:
	xs := []int{8, 6, 8, 2, 4, 4, 5, 9}
	
	// For loop is the only generic way to traverse slices, you we have to write the following:
	index := map[int]struct{}{}
	for _, x := range xs {
		index[x] = struct{}{}
	}
	
	// We can "easily" acquire the deduped slice by using a for loop again...
	deduped := []int{}
	for k, _ := range index {
		deduped = append(deduped, k)
	}
	// Hooray, we can use the 'deduped' slice!
	fmt.Println(deduped)
}
{% endhighlight %}
(<a href="http://play.golang.org/p/Mo_ZfbJNJF">playground link</a>)

For those who are not familiar with the concept of generics, here is a though experiment: let's refactor that bit of information by moving it out to a function:

{% highlight go %}
package main

func deduper(xs []int) []int {
	index := map[int]struct{}{}
	for _, x := range xs {
		index[x] = struct{}{}
	}
	deduped := []int{}
	for k, _ := range index {
		deduped = append(deduped, k)
	}
	return deduped
}
{% endhighlight %}

Uh-oh: now our method only works on maps - our for loops would be still generic, but the function definition forces us to tell the type of the input argument. It is an int slice. If somehow we could tell the compiler that we don't care what kind of slice it is!

You may ask - what if we use the interface{} interface type? It is a bit ugly, but it works! Let's try that!

{% highlight go %}
func deduper(xs []interface{}) []interface{} {
	index := map[int
{% endhighlight %}

Uh-oh again. We even had to stop typing. We can not use the empty interface as our key in the map... To be able to use something as a map key the members of that type must be comparable (<a href="http://golang.org/ref/spec#Comparison_operators">http://golang.org/ref/spec#Comparison_operators</a>). Empty interfaces are not comparable, since they can represent non-comparable types! This way we can forgot our neat implementating which reuses the idempotent nature of setting keys of a map!

Let's look for an other approach - surely the Go authors have paved the way for us. Let's take a look at the <a href="http://golang.org/pkg/sort/">sort package</a>. We see a quite descriptively named <a href="http://golang.org/pkg/sort/#Interface">sort.Interface</a> type there:

{% highlight go %}
type Interface interface {
        // Len is the number of elements in the collection.
        Len() int
        // Less reports whether the element with
        // index i should sort before the element with index j.
        Less(i, j int) bool
        // Swap swaps the elements with indexes i and j.
        Swap(i, j int)
}
{% endhighlight %}

This would be all good, but no methods can be defined on builtin types! Don't worry! The <a href="http://golang.org/pkg/sort/#IntSlice">IntSlice</a> type comes for the rescue! We only have to typecast our []int into an IntSlice and we can use all the functions written by other very smart people. But let's go back to the deduping function. Let's try to use the sort.Interface to write our own deduping function. After all, we can compare elements of a slice with it.

{% highlight go %}
package main

import "sort"

func dedupe(xs sort.Interface) {
	for _, v := range xs {
	
	}
}
{% endhighlight %}
(<a href="http://play.golang.org/p/grnXYt76pE">playground link</a>)

Before finishing our function we realize one thing - we can not iterate over the sort.Interface:

{% highlight go %}
prog.go:6: type sort.Interface is not an expression
prog.go:7: cannot range over xs (type sort.Interface)
 [process exited with non-zero status]
{% endhighlight %}

The main problem here is that we've lost information - the sort.Interface is not a slice anymore. We've lost the ability to iterate over it the moment we created an interface out of it - which was not our intention at all. We only had to do that due to Go's inability to express concepts in a generic way. Interfaces are very one dimensional. We can not build on top of them. In an expressive language, like haskell's, we can say: "given a type 'a' which elements can be compared against each other to see if they are equivalent, we can write a function which removes duplicates from a list (slice) of these elements".

That function, in the Haskell standard library, is Data.List.nub:

{% highlight haskell %}
> import Data.List
> :t nub
nub :: Eq a => [a] -> [a]
> nub [8, 6, 8, 2, 4, 4, 5, 9]
[8,6,2,4,5,9]
> nub ['a', 'b', 'a', 'c']
['a', 'b', 'c']
{% endhighlight %}

The above snippets illustrates how nub works with a list of anything, as long as anything is an instance of the Eq typeclass - ie. there is a defined way to compare them on grounds of equality.

For those who are bothered about the exponential algorithmic complexity of nub, let's recreate our efficient (albeit nongeneric) Go solution in Haskell.

Nub requires the elements of the list to be instances of the Eq typeclass, that is why it performs soo poorly. If we are stricter and require the elements to be instances of Ord typclass, as is the case with Go's maps indices, we can write a more efficient function, which pretty much does the same thing as the Go code snippet above - puts the list elements to a map and then converts back to a list:

{% highlight haskell %}
import qualified Data.Map as M
let nubWell xs = map (\(k, _) -> k) . M.toList . M.fromList $ map (\x -> (x, ())) xs
> :t nubWell
nubWell :: Ord b => [b] -> [b]
> nubWell [8, 6, 8, 2, 4, 4, 5, 9]
[2,4,5,6,8,9]
{% endhighlight %}

Just to show you that it actually works on any instance of the Ord typeclass, let's nub a list of strings:

{% highlight haskell %}
> nubWell ["generics", "are", "useful", "useful", "it", "is", "a", "useful", "fact"]
["a","are","fact","generics","is","it","useful"]
{% endhighlight %}

Please note that our version of nub is not order preserving - neither is the Go version due to the randomized traversal of maps. 

<a href="https://groups.google.com/forum/#!topic/golang-nuts/-pqkICuokio">This thread</a> discusses the same problem, without finding a nice solution - because there isn't any - the type system is just not expressive enough.

#### Standard list operations

In go we are forced to use iteration as our tool to traverse slices and maps - because iteration is polymorphic builtin construct. The verbosity of iteration is mind-bogging, combined with appending, ifs and other constructs makes the code way more involved than it should be.

Let's observe a scenario when we want to remove elements from a slice:

{% highlight go %}
xs := []int{0,1,2,3,4,5,6}

// Remove elements smaller than 4
smallerThanFour := []int{}
for _, x := range xs {
	if x < 4 {
		smallerThanFour = append(smallerThanFour, x)
	}
}
{% endhighlight %}
(<a href="http://play.golang.org/p/JU-yUKTEZS">playground link</a>)

{% highlight haskell %}
> let xs = [0..6]
> filter (<4) xs
[0,1,2,3]
{% endhighlight %}

Or if we want to remove an element from a slice

{% highlight go %}
xs := []int{0,1,2,3,4,5,6}
noFiveHere := []int{}
for _, x := range xs {
	if x != 5 {
		noFiveHere = append(noFiveHere, x)
	}
}
{% endhighlight %}

{% highlight haskell %}
> let xs = [0..6]
> filter (/=5) xs
[0,1,2,3,4,6]
{% endhighlight %}

These operations would be possible in Go with the help of generics... although the type signature of the lambda would have to be stated explicitly, since Go <a href="http://programmers.stackexchange.com/questions/253558/type-inference-in-golang-haskell">does not support type inference</a> (not the rather powerful Hindler-Milner anyway).

#### Empty interfaces everywhere

Without generics support, it is impossible to create a generic type parametrized over an other one. It is especially painful when dealing with container-like types, sets, trees etc. 

The type signature becomes littered with interface{}-s, decreasing the readabilty and the type safety of the language.

Take a look at the following package: <a href="http://golang.org/pkg/container/list">The conatiner/list package in the standard library</a>

The type signature of the List type is the following:

{% highlight go %}
type List
    func New() *List
    func (l *List) Back() *Element
    func (l *List) Front() *Element
    func (l *List) Init() *List
    func (l *List) InsertAfter(v interface{}, mark *Element) *Element
    func (l *List) InsertBefore(v interface{}, mark *Element) *Element
    func (l *List) Len() int
    func (l *List) MoveAfter(e, mark *Element)
    func (l *List) MoveBefore(e, mark *Element)
    func (l *List) MoveToBack(e *Element)
    func (l *List) MoveToFront(e *Element)
    func (l *List) PushBack(v interface{}) *Element
    func (l *List) PushBackList(other *List)
    func (l *List) PushFront(v interface{}) *Element
    func (l *List) PushFrontList(other *List)
    func (l *List) Remove(e *Element) interface{}
{% endhighlight %}

All the functions that are dealing with the elements of the List are defined with empty interfaces. That could be completely avoided if the List type could be parametrized over other types (like the builtin types map and slice can be. If you want to read more about these types, which seem to require other types to produce a 'final type', start <a href="http://en.wikipedia.org/wiki/Kind_%28type_theory%29">here<a>).

### Lack of algebraic data types

<a href="http://en.wikipedia.org/wiki/Tagged_union">Algebraic data types</a>, or sum types are basically types which can represent a certain number of fixed types.

So while in a struct the different fields are present in the same type, an ADT may contain only one of those fields at a time. An example would be:

#### Nondescriptive types

{% highlight haskell %}
data Tree = Leaf
          | Node Int Tree Tree
{% endhighlight %}

The same concept in Go would be expressed as:

{% highlight go %}
type Tree struct {
	Int int
	Left *Tree
	Right *Tree
	Leaf *Leaf
}
{% endhighlight %}

All the possible values would be present at all times - even if they are not being used at all. This gets messy quiet quickly.

#### A rainbow of grey, grey, and grey

Apart from readability, type safety suffers as well. The moment we need something to be able to hold more than one type, suddenly it becomes

#### Multiple return values

Multiple return values are really just a special case of <a href="http://en.wikipedia.org/wiki/Tuple">tuples</a>, except they are much less composable. One might argue, they are intentionally hard to compose, so it makes ignoring returned errors harder. However that would be a pretty bad argument. Let's examine the following code snippet:

{% highlight go %}
package main

import "os"

func main() {
	file, err = os.Open("file.txt")
	if err == nil {
		return
	}
	file.Chmod(777)
}
{% endhighlight %}
(<a href="http://play.golang.org/p/oVGfd-e2w1">playground link</a>)

The compiler happily accepts this while this is clearly a bug. Tools like Go vet may catch the errors - however - that is just band aid. Guaranteeing code correctess is the job of the compiler, a tool like Go vet may utilize ad hoc solutions to detect certain specific problems with the code but that solution will never be as coherent and all encompassing as a sufficiently expressive type system can be.

With the help of ADTs (and pattern matching) this problem can be easily detected (more on this later).

### Methods, interfaces, and other minor annoyances

#### Interfaces are implemented implicitly

Interfaces in Go are implemented implicitly, which means if someone happens to come along and creates an interface with a method which your type already has your type immediately implements that interface - wether you like it or not. While this happens rather rarely in practice (in my experience), I do not understand the motivation behind this design decision - usually Go prefers explicitness over implicitness, why the exception here? When writing a methods which is implementing an interface one has a specific interface in mind anyway - then why lose this information?

#### No way to define methods on foreign types

The biggest reason why go had to abandon the very concept of defining methods on foreign types is because interfaces are implemented implicitly - a decision which increases Go's inconvenience factor severely. It saves a bit of typing in the short term but has utterly devastating consequences in the long term: one has to created new named type

Interestingly, Go already has syntax which would be well suited to this (not like syntax is an important matter when it comes to issues like this), as it is noted in <a href="https://groups.google.com/forum/#!topic/golang-nuts/Hbxekd9g09c">this thread<a> by Rasmus Schultz

> am I missing something here?
>
> just learning Go, and I was under the impression that one of the key reasons for the "detached" method-declaration syntax, was that you would be able to extend somebody else's type with new methods required by your program, in a "non-invasive" manner.
>
> so you can't add methods to a type unless it was declared in the local package, is that right?
>
> what exactly is the point of the detached method-declarations then?

#### No type aliases

Sometimes it is good to have type aliases which are basically macros doing simple string replace in the source, to increase readability, for example the <a href="https://github.com/go-mgo/mgo">excellent MongoDB driver, mgo</a> defines a convenience type M:

{% highlight go %}
type M map[string]interface{}
{% endhighlight %}

This is just to avoid typing map[string]interface{} when defining BSON documents. Compare:

{% highlight go %}
customer := map[string]interface{
	"id": "42"
	"cats": []interface{
		map[string]interface{
			"Name": "Joe",
			"Age": 2,
		},
		map[string]interface{
			"Name": "Joline",
			"Age": 2,
		}
	}
}
{% endhighlight %}

vs:

{% highlight go %}
customer := M{
	"id": "42"
	"cats": []interface{
		M{
			"Name": "Joe",
			"Age": 2,
		},
		M{
			"Name": "Joline",
			"Age": 2,
		}
	}
}
{% endhighlight %}

Unfortunately, in Go, M and map[string]interface{} become completely separate types - we have to typecast from one type to satisfy the compiler.

#### Type assertions

Type assertions are completely type unsafe - a function should never ever make assumptions about the underlying type of an interface type - that completely defies the purpose of interfaces to begin with.

### Conclusion

To be finished.
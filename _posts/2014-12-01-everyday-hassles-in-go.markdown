---
layout: post
title:  "Everyday hassles in Go"
date:   2014-12-01
---

Go became a reliable ally of mine during the past years. I use it in my day job and for side projects alike. I quite like the Go ecosystem, the tooling is great, from go fmt to race detector one can feel that the authors have done their share of coding in the wild. The standard library is very comprehensive (especially considering the age of the language), the documentation is top notch. The language itself though, could be improved a lot by borrowing battle tested ideas from more modern ones.

Unfortunately, a lot of Go programmers are coming from untyped languages, which means they haven't yet acquired the taste for sufficiently expressive type systems, thus they may not know about alternative approaches. A snarky person might say, they suffer from the <a href="http://www.paulgraham.com/avg.html">blub paradox</a>.

This lack of perspective in the Go community hinders the progress of the language - people do not exert enough force toward the authors (not like they seem to be crowd pleasers anyway) to better the language. While I am grateful for Go as a tool, I am slightly worried about its potential educational effect - or the lack of it. Given it is backed by Google - due to the hype and exposure that brings - even design failures will be accepted as 'the way to do it' by a large number of people. People like the authors of Go have an immense responsibility when it comes to improving our industry as a whole.

Here are the analysis of some of the features (or the lack of them) I consider unfortunate, with use cases and accompanying code. The examples may be quite arbitrary. Most of the difficulties listed here could be fixed by a relatively small number of changes (<a href="http://en.wikipedia.org/wiki/Pareto_principle">the 80/20 rule?</a>).

### Lack of generics

Generics are well supported by a wide range of languages, and is one of the most requested lacking features of Go. The usual response from the Go authors is that implementing generics will either slow down compilation, the code, or the programmer. While that is almost a believable reason, the funnier part of their reasoning is (excerpt from FAQ):

> Go's built-in maps and slices, plus the ability to use the empty interface to construct containers (with explicit unboxing) mean in many cases it is possible to write code that does what generics would enable, if less smoothly.

(<a href="https://golang.org/doc/faq#generics">link<a>)

Well, how less smoothly? Let's investigate.

(Quite a good chunk of this blog post will be dedicated to help people who never used generics to develop an intuition why generics are useful. If you are not interested in this part, scroll down a bit.)

#### Bye-bye code reuse

A rather surprising set of functionality is missing from the standard library: a type of 'bread and butter' code which deals with <a href="http://hackage.haskell.org/package/base-4.7.0.1/docs/Data-List.html">simple but often encountered scenarios</a> that no programmer should implement, because it is simply a waste of brainpower cycles - there is no gain to be won by reimplementing - let's say - deduping elements in a slice.

##### Deduping slice elements

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

For those who are not familiar with the concept of generics, here is a thought experiment: let's refactor that bit of code by moving it out to a function:

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

Uh-oh: now our method only works on int slices - our for loops would be still generic, but the function definition forces us to tell the type of the input argument. It is an int slice. If somehow we could tell the compiler that we don't care what kind of slice it is!

You may ask - what if we use the interface{} interface type? It is a bit ugly, but it works! Let's try that!

{% highlight go %}
func deduper(xs []interface{}) []interface{} {
	index := map[int
{% endhighlight %}

Uh-oh again. We even had to stop typing. We can not use the empty interface as our key in the map... To be able to use something as a map key the members of that type must be comparable (<a href="http://golang.org/ref/spec#Comparison_operators">http://golang.org/ref/spec#Comparison_operators</a>). Empty interfaces are not comparable, since they can represent non-comparable types! This way we can forget our neat implementation which reuses the idempotent nature of setting keys of a map!

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

The main problem here is that we've lost information - the sort.Interface is not a slice anymore. We've lost the ability to iterate over it the moment we created an interface out of it. We could provide all the operations which can be done on a slice in an interface, in the following way:

{% highlight go %}
type Slice interface{
	Get(i int) interface{}
	// ...
{% endhighlight %}

We pretty much had to stop at the very first method, we have to abuse the interface{} type everywhere. If only we could write something like the following:

{% highlight go %}
type Slice a interface {
	Get(i int) a
	Len() int
	Append(a) []a
}
{% endhighlight %}

That way, our compiler would know that the Get method of a Slice Int returns an int, the Get method of a Slice String returns a string etc.

In an expressive type system, like haskell's, we can do exactly that. The deduping example can be stated the following way: "given a type 'a' which elements can be compared against each other to see if they are equivalent, we can write a function which removes duplicates from a list of these elements".

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

For those who are bothered about the quadratic algorithmic complexity of nub, let's recreate our efficient (albeit nongeneric) Go solution in Haskell.

Nub requires the elements of the list to be instances of the Eq typeclass, that is why it performs so poorly. If we are stricter and require the elements to be instances of Ord typeclass, as is the case with Go's maps indices, we can write a more efficient function, which pretty much does the same thing as the Go code snippet above - puts the list elements to a map and then converts back to a list:

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

#### Flow disruption

Perhaps the most elusive, but rather destructive aspect of the lack of generics in Go is how the language forces the user to go into uninteresting details while expressing ideas, disrupting the programmer's flow. We don't have to venture far to see examples of this - we can stay at the topic of list operations, as we did in the previous example.

##### Standard list operations

In go we are forced to use iteration as our tool to traverse slices and maps - because iteration is a polymorphic builtin construct. The verbosity of iteration is mind-bogging, combined with appending, ifs and other constructs makes the code way more involved than it should be.

###### Filter

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

###### Map

Increase every number by in in an array:

{% highlight go %}
package main

import "fmt"

func incByOne(xs []int) xs {
	ret := []int{}
	for _, v := range ints {
		ret = append(ret, v+1)
	}
	return ret
}

func main() {
	ints := []int{1,2,3,4,5,6,7,8,9,10}
	fmt.Println(incByOne(ints))
}
{% endhighlight %}
(<a href="http://play.golang.org/p/sykA1S07-A">playground link</a>)

{% highlight haskell %}
> let incByOne xs = map (+1) xs
> incByOne [1..10]
[2,3,4,5,6,7,8,9,10,11]
{% endhighlight %}

###### Folding

Summing numbers:

{% highlight go %}
package main

import "fmt"

func mySum(xs []int) int {
	ret := 0
	for _, v := range xs {
		ret += v
	}
	return ret
}

func main() {
	fmt.Println(mySum([]int{1,2,3,4,5}))
}
{% endhighlight %}
(<a href="http://play.golang.org/p/gspWQYH1AT">playground link</a>)

{% highlight haskell %}
> let mySum xs = foldl (+) 0 xs
> mySum [1..5]
15
{% endhighlight %}

These operations would be possible in Go with the help of generics... although the type signature of lambdas would have to be stated explicitly, since Go <a href="http://programmers.stackexchange.com/questions/253558/type-inference-in-golang-haskell">does not support type inference</a> (not the rather powerful <a href="http://en.wikipedia.org/wiki/Hindley%E2%80%93Milner_type_system">Hindley-Milner</a> one anyway).

#### The billion dollar mistake

Nil pointers, as implicit valid values for all pointer types, and interfaces are one of the most frequent sources of bugs. The problem lies in the fact that one can dereference a pointer without proving that it actually holds a non-nil value, eg.:

{% highlight go %}
file, _ = os.Open("file.txt")
file.Chmod(777)
{% endhighlight %}

Is perfectly valid as far as the compiler is concerned. Meanwhile, using Option or Maybe types the same mistake can not be done:

{% highlight haskell %}
> case find (==2) [3..5] of
>	Just x 	-> print "Found! Yay :)"
>   Nothing -> print "Not found :("
"Not found :("
{% endhighlight %}

The different cases must be handled explicitly, forcing the programmer to think about both of them.

#### Empty interfaces everywhere

Without generics support, it is impossible to create a generic type parametrized over an other one. It is especially painful when dealing with container-like types, sets, trees etc. 

The type signature becomes littered with interface{}-s, decreasing the readabilty and the type safety of the language.

Take a look at the following package: <a href="http://golang.org/pkg/container/list">The container/list package in the standard library</a>

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

All the functions that are dealing with the elements of the List are defined with empty interfaces. That could be completely avoided if the List type could be parametrized over other types (like the builtin types map and slice can be). If you want to read more about those types, which seem to require other types to produce a 'final type', start <a href="http://en.wikipedia.org/wiki/Kind_%28type_theory%29">here<a>).

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

All the possible values would be present at all times - even if they are not being used at all. This gets messy quite quickly.

#### A rainbow of grey, grey, and grey

Apart from readability, type safety suffers as well. The moment we need something to be able to hold more than one type, suddenly we are forced to use interface{}

A very good example is the definition of a JSON value. In Haskell, the type reads clearly:

{% highlight haskell %}
-- | A JSON \"object\" (key\/value map).
type Object = HashMap Text Value

-- | A JSON \"array\" (sequence).
type Array = Vector Value

-- | A JSON value represented as a Haskell value.
data Value = Object !Object
           | Array !Array
           | String !Text
           | Number !Number
           | Bool !Bool
           | Null
             deriving (Eq, Show, Typeable)
{% endhighlight %}
(<a href="http://hackage.haskell.org/package/aeson-0.6.1.0/docs/src/Data-Aeson-Types-Internal.html#Value">link</a>)

A JSON Value is an object, a string, a number, a bool or a Null value. What is the best we can do in Go?

{% highlight go %}
type JSON map[string]interface{}
{% endhighlight %}

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

The compiler happily accepts this while this is clearly a bug. Tools like Go vet may catch the errors - however - that is just band aid. Guaranteeing code correctness is the job of the compiler, a tool like Go vet may utilize ad hoc solutions to detect certain specific problems with the code but that solution will never be as coherent and all encompassing as a sufficiently expressive type system can be.

### Interfaces are misdesigned

#### Interfaces are implemented implicitly

Interfaces in Go are implemented implicitly, which means if someone happens to come along and creates an interface with a method which your type already has your type immediately implements that interface - wether you like it or not. While this happens rather rarely in practice (in my experience), the implicit nature of this feels out of place - usually Go prefers explicitness over implicitness, why the exception here? 

When writing a methods which is implementing an interface one has a specific interface in mind anyway - then why lose this information? In fact, readability suffers - one has no way to know which interfaces a type implements. It also prevents us to state how non local types implement an interface...

#### No way to define how a foreign type implements an interface

Who knows better how a certain type satisfies an interface than the person who defined the interface? Requiring a package to be modified so a type can implement as interface severely hinders extensibility.

Interestingly, Go already has syntax which would be well suited to this (not like syntax is an important matter when it comes to issues like this), as it is noted in <a href="https://groups.google.com/forum/#!topic/golang-nuts/Hbxekd9g09c">this thread</a> by Rasmus Schultz.

> I was under the impression that one of the key reasons for the "detached" method-declaration syntax, was that you would be able to extend somebody else's type with new methods required by your program, in a "non-invasive" manner.

#### Lack of deriving

Default implementations for certain interfaces should be provided. How can Go print out our structs or maps correctly? We don't know. It is done with reflection, which is runtime concept and sidesteps the type system entirely. The ad hoc, edge case laden nature of Go shows here again. For those who are unfamiliar with the concept:

If we define a record (struct in Go land), and we want to print an instance of it, we get a compile error.

{% highlight haskell %}
> data Customer = Customer { name :: String, age :: Int }
> print $ Customer "Joe" 14

<interactive>:3:1:
    No instance for (Show Customer) arising from a use of ‘print’
    In the expression: print
    In the expression: print $ Customer "Joe" 14
    In an equation for ‘it’: it = print $ Customer "Joe" 14
{% endhighlight %}

The compiler error tells us that our Customer type is not an instance of the Show typeclass - in Go terminology, the Customer type does not implement the Show interface. Inspecting the type signature of print we see it indeed requires any element passed to it to be an instance of Show:

{% highlight haskell %}
> :t
print :: Show a => a -> IO ()
{% endhighlight %}

Using derive, we can get default implementations for free:

{% highlight haskell %}
> data Customer = Customer { name :: String, age :: Int } deriving Show
> print $ Customer "Joe" 14
Customer {name = "Joe", age = 14}
{% endhighlight %}

Also interesting how our Customer type 'officially' became an instance of the Show typeclass:

{% highlight haskell %}
Prelude> :i Customer
data Customer = Customer {name :: String, age :: Int}
  	-- Defined at <interactive>:7:1
instance Show Customer -- Defined at <interactive>:7:66
{% endhighlight %}

So while the implementation is 'magic', in all other regards these type classes behave the same way as they would be user implemented.

#### Type assertions

Type assertions are completely type unsafe - a function should never ever make assumptions about the underlying type of an interface type - that completely defies the purpose of interfaces to begin with.

### Other minor annoyances

#### General inelegance

##### Superflous syntax

Almost everywhere superflous brackets of all types can be found. What is the purpose of:

{% highlight go %}
map[string]interface{}
{% endhighlight %}

vs 

{% highlight go %}
Map String Any
{% endhighlight %}
(hypothetical Haskell equivalent)

? Go got rid of some of these (for loop, if requires no parentheses), but there are still plenty.

##### Operators should be functions

The only reason operators are not functions is because the type system is not sophisticated enough to describe them.

#### No type aliases

Sometimes it is good to have type aliases which are basically macros doing simple string replace in the source, to increase readability, for example the excellent MongoDB driver, <a href="https://github.com/go-mgo/mgo">mgo</a> defines a convenience type M:

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

### Conclusion

Go focuses on simplicity, but the lack of strong foundations introduces edge cases in a lot of places, making the language more complex and less useful than it could be. But even with those edge cases, Go is still easier to learn than most other typed languages, and that is probably its the biggest selling point. It very much feels like a dynamic language with a very minimalistic type system. This has the positive effect of introducing people who were exposed only to dynamic languages to the world of type system due to its approachability. However, this simplicity may very well adversely affect the Go programmers by accepting suboptimal solutions.

The support and hype it receives might be better spent on more novel languages with bigger potential to better our industry.

### Appendix

- <a href="https://news.ycombinator.com/item?id=8814202">HN thread</a>
- <a href="http://www.reddit.com/r/programming/comments/2qtgwm/everyday_hassles_in_go_xpost_from_rgolang/">r/programming Reddit thread</a>
- <a href="http://www.reddit.com/r/golang/comments/2qtaf2/everyday_hassles_in_go/">r/golang Reddit thread</a>
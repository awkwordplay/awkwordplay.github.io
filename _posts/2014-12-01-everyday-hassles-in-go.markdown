---
layout: post
title:  "Everyday hassles in Go"
date:   2014-12-01
---

### Lack of generics

Perhaps the most irritating aspect of the lack of generics in Go is how the language forces me to go into uninteresting details while expressing my ideas, distrupting my flow. Unfortunately a lot of Go programmers are coming from untyped languages, which means they haven't yet acquired the taste for sufficiently expressive type systems. A snarky person might say, they suffer from the <a href="http://www.paulgraham.com/avg.html">blub paradox</a>. For them, here are a couple of examples why generics make sense.

(The following examples will be quite arbitrary.)

#### Deduping slice elements

Deduping elements of a slice happens the following way in go:

{% highlight go %}
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
{% endhighlight %}

Take a language with generics support, Haskell, this can be abstracted away in a function,  Data.List.nub works wonders (in ghci:)

{% highlight haskell %}
> import Data.List
> :t nub
> nub [8, 6, 8, 2, 4, 4, 5, 9]
[8,6,2,4,5,9]
> nub ['a', 'b', 'a', 'c']
['a', 'b', 'c']
{% endhighlight %}

(For those who are bothered about the exponential algorithmic complexity of nub, nub require s the elements of the list to be instances of the Eq typeclass, that is why it performs soo poorly. If we are stricter and require the elements to be instances of Ord typclass, as is the case with Go's maps indices, we can write a more efficient function, which pretty much does the same thing as the Go code snippet above - puts the list elements to a map and then converts back to a list:)

{% highlight haskell %}
import qualified Data.Map as M
let nubWell xs = map (\(k, _) -> k) . M.toList . M.fromList $ map (\x -> (x, ())) xs
> :t nubWell
nubWell :: Ord b => [b] -> [b]
> nubWell [8, 6, 8, 2, 4, 4, 5, 9]
[2,4,5,6,8,9]
{% endhighlight %}

Just to show you that it actually works on any instance of the Ord typeclass:

{% highlight haskell %}
> nubWell ["generics", "are", "useful", "useful", "it", "is", "a", "useful", "fact"]
["a","are","fact","generics","is","it","useful"]
{% endhighlight %}

(Please not that our version of nub is not order preserving - neither is the Go version due to the randomized traversal of maps.)

<a href="https://groups.google.com/forum/#!topic/golang-nuts/-pqkICuokio">This thread</a> discusses the same problem.

#### Standard list operations

The verbosity of iteration is mind-bogging, combined with appending ifs and other constructs makes the code way more involved than it should be.

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

#### 

#### Empty interfaces everywhere

Without generics support, it is impossible to create a generic type parametrized over an other one. It is especially painful when dealing with container-like types, sets, trees etc. 

The type signature becomes littered with interface{}-s, decreasing the readabilty and the type safety of the language.

Take a look at the following package: <a href="https://github.com/deckarep/golang-set">https://github.com/deckarep/golang-set</a>

The type signature of the Set type is the following:

{% highlight go %}
type Set interface {
	Add(i interface{}) bool
	Cardinality() int
	Clear()
	Clone() Set
	Contains(i ...interface{}) bool
	Difference(other Set) Set
	Equal(other Set) bool
	Intersect(other Set) Set
	IsSubset(other Set) bool
	IsSuperset(other Set) bool
	Iter() <-chan interface{}
	Remove(i interface{})
	String() string
	SymmetricDifference(other Set) Set
	Union(other Set) Set
	PowerSet() Set
	CartesianProduct(other Set) Set
	ToSlice() []interface{}
}
{% endhighlight %}

All the functions that are dealing with the elements of the Set are defined with empty interfaces. It's void pointers all over again.

### Lack of algebraic data types

#### A rainbow of grey, grey, and grey

{% highlight go %}
{% endhighlight go %}
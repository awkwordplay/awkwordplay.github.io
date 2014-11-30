---
layout: post
title:  "Everyday hassles in Go"
date:   2014-11-30
---

Mayhaps the most irritating aspect of the lack of generics in Go is how the language forces me to go into uninteresting details while expressing my ideas. Unfortunately a lot of Go programmers are coming from untyped languages, which means they haven't yet acquired the taste for sufficiently expressive type systems. A snarky person might say, they suffer from the [url=http://www.paulgraham.com/avg.html]blub paradox[/url]. For them, here are a couple of examples why generics make sense.

Working with slices
===

Deduping elements of a slice happens the following way in go:

{% highlight go %}
// Given the following list:
m := []int{8, 6, 8, 2, 4, 4, 5, 9}
// For loop is the only generic way to traverse slices, you we have to write the following:
index := map[int]struct{}{}
for _, v := range m {
	index[v] = struct{}{}
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
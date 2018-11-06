---
layout: post
title: "Anatomy of semigroups and monoids"
author: "Myself"
date: 2018-11-06
catorgories: [anatomy-atlas]
tags: [Scala, Functional programming]
---

I will try to group here, in an anatomy atlas, basic notions of functional programming that I find myself explaining often lately into a series of articles.

The idea here is to have a place to point people needing explanations and to increase my own understanding of these subjects by trying to explain them the best I can.
I'll try to focus more on making the reader feel an intuition, a feeling about the concepts rather than on the perfect, strict correctness of my explanations.

- Part 1: [Anatomy of functional programming]({{ site.baseurl }}{% post_url 2018-10-08-fp %})
- Part 2: [Anatomy of an algebra]({{ site.baseurl }}{% post_url 2018-10-06-algebras %})
- Part 3: [Anatomy of a type class]({{ site.baseurl }}{% post_url 2018-10-05-typeclasses %})
- Part 4: [Anatomy of semi groups and monoids]({{ site.baseurl }}{% post_url 2018-11-06-semi-monoid %})
- Part 5: Anatomy of functors, applicatives and monads - Yet to come !
- Part 6: Anatomy of the tagless final encoding - Yet to come !

# What is a _semigroup_ ?

## General definition

_Semigroup_ (and _monoid_, you'll see later) is a complicated word for a __really__ simple concept.
We'll cover quickly _semigroups_ and we'll explain longer _monoids_ since they are strongly related.

_Wikipedia's_ definition is:

> In mathematics, a _semigroup_ is an algebraic structure consisting of a set together with an associative binary operation.

Ok, that sounds a bit abstract, let's try to re-phrase it with programming terms:

In the context of programming, a _semigroup_ is composed of two things:

1. A type `A`
2. An associative operation combining two values of type `A` into a value of type `A`, let's call it `combine`
    - That would be a _function_ with a type signature: `(A, A) => A`
    - Which is associative, meaning that the order in which you combine elements together (where you decide to put your parenthesis) does not matter
        - `combine(combine(a1, a2), a3) == combine(a1, combine(a2, a3))` with `a1`, `a2`, `a3` values of type `A`

Then it is said that __`A` forms a _semigroup_ under `combine`__.

## Some examples

### Integer under addition

- type: `Int`
- operation: `+`

Indeed,

- `+` type here is: `(Int, Int) => Int`
- `+` is associative `(20 + 20) + 2 == 20 + (20 + 2)`

Integers form a _semigroup_ under addition.

### Boolean under OR

- type: `Boolean`
- operation: `||`

Indeed,

- `||` type here is: `(Boolean, Boolean) => Boolean`
- `||` is associative `(true || false) || true == true || (false || true)`

Booleans form a _semigroup_ under OR.

### List under list concatenation

- type: `List[A]`
- operation: `++`

Indeed,

- `++` type here is: `(List[A], List[A]) => List[A]`
- `++` is associative `(List(1, 2) ++ List(3, 4)) ++ List(5, 6) == List(1, 2) ++ (List(3, 4) ++ List(5, 6))`

### More examples !

- Integers under multiplication
- Booleans under AND
- String under concatenation
- A LOT more.

We'll now explore _monoids_ since they are a "upgraded" version of _semigroups_.

# What is a _monoid_ ?

## General definition

Given the definition of a _semigroup_, the definition of a _monoid_ is pretty straight forward:

> In mathematics, a _monoid_ is an algebraic structure consisting of a set together with an associative binary operation and an identity element.

Which means that a _monoid_ is a _semigroup_ plus an identity element.

In our programming terms:

In the context of programming, a _monoid_ is composed of two things:

1. A _semigroup_:
    - A type `A`
    - An associative operation combining two values of type `A` into a value of type `A`, let's call it `combine`
2. An identity element of type `A`, let's call it `id`, that has to obey the following laws:
    - `combine(a, id) == a` with `a` a value of type `A`
    - `combine(id, a) == a` with `a` a value of type `A`

Then it is said that __`A` forms a _monoid_ under `combine` with identity element `id`__.

## Some examples

We could take our _semigroups_ examples here and add their respective identity elements:

- Integer under addition
    - With identity element `0`:
        - `42 + 0 = 42`
        - `0 + 42 = 42`
- Boolean under OR
    - With identity element `false`:
        - `true || false == true`, `false || true == true`
        - `false || false == false`
- List under list concatenation
    - With identity element `Nil` (empty List):
        - `List(1, 2, 3) ++ Nil == List(1, 2, 3)`
        - `Nil ++ List(1, 2, 3) == List(1, 2, 3)`

Whenever you have an identity element for your _semigroup_'s type and `combine` operation that holds the identity laws, then you have a _monoid_ for it.

But be careful, there are some _semigroups_ which are not _monoids_:

Tuples form a _semigroup_ under first (which gives back the tuple's first element).

- type: `Tuple2[A, A]`
- operation: `first` (`def first[A](t: Tuple2[A, A]): A = t._1`)

Indeed,

- `first` type here is: `Tuple2[A, A] => A`
- `first` is associative `first(Tuple2(first(Tuple2(a1, a2)), a3)) == first(Tuple2(a1, first(Tuple2(a2, a3))))` with `a1`, `a2`, `a3` values of type `A`

But there is no way to provide an identity element `id` of type `A` so that:

- `first(Tuple2(id, a)) == a` and `first(Tuple2(a, id)) == a` with `a` a value of type `A`

# What the hell is it for ?

_Monoid_ is a functional programming constructs that __embodies the notion of combining "things" together__, often in order to reduce "things" into one "thing". Given that the combining operation is associative, it can be __parallelized__.

And that's a __BIG__ deal.

As a simple illustration, this is what you can do, absolutely fearlessly when you know your type `A` forms a _monoid_ under `combine` with identity `id`:

- You have a huge, large, massive list of `A`s that you want to reduce into a single `A`
- You have a cluster of N nodes and a master node
- You split your huge, large, massive list of `A`s in N sub lists
- You distribute each sub list to a node of your cluster
- Each node reduce its own sub list by `combining` its elements 2 by 2 down to 1 final element
- They send back their results to the master node
- The master node only has N intermediary results to `combine` down (in the same order as the sub lists these intermediary results were produced from, remember, associativity !) to a final result

You successfully, without any fear of messing things up, parallelized, almost for free, a reduction process on a huge list thanks to _monoids_.

Does it sound familiar ? That's naively how fork-join operations works on Spark ! Thank you _monoids_ !

# How can we encode them in Scala ?

_Semigroups_ and _monoids_ are encoded as [type classes]({{ site.baseurl }}{% post_url 2018-10-05-typeclasses %}).

We are gonna go through a simple implementation example, you should never have to do it by hand like that since everything we'll do is provided by awesome FP libraries like [Cats](https://github.com/typelevel/cats) or [Scalaz](https://github.com/scalaz/scalaz).

Here are our two type classes:

```scala
trait Semigroup[S] {
    def combine(s1: S, s2: S): S
}

trait Monoid[M] extends Semigroup[M] {
    val id: M
}
```

And here is my business domain modeling:

```scala
type ItemId = Int
case class Sale(items: List[ItemId], totalPrice: Double)
```

I want to be able to combine all my year's sales into one big, consolidated, sale.

Let's define a _monoid_ type class instance for `Sale` by defining:

- `id` being an empty `Sale` which contains no item ids, and 0 as `totalPrice`
- `combine` as concatenation of item id lists and addition of `totalPrice`s

```scala
implicit val saleMonoid: Monoid[Sale] = new Monoid[Sale] {
    override val id: Sale                          = Sale(List.empty[ItemId], 0)
    override def combine(s1: Sale, s2: Sale): Sale = Sale(s1.items ++ s2.items, s1.totalPrice + s2.totalPrice)
}
```

Then I can use a lot of existing tooling, generic functions, leveraging the fact that the types they are working on are instances of _monoid_.

`combineAll` (which is also provided by _Cats_ or _Scalaz_) is one of them and permit to, generically, combine all my sales together for free !

```scala
def combineAll[A](as: List[A])(implicit M: Monoid[A]): A = {
    def accumulate(accumulator: A, remaining: List[A]): A = remaining match {
        case Nil          ⇒ accumulator
        case head :: tail ⇒ accumulate(M.combine(accumulator, head), tail)
    }
    accumulate(M.id, as)
}

val sales2018: List[Sale] = List(Sale(List(0), 32), Sale(List(1), 10))
val totalSale: Sale       = combineAll(sales2018) // Sale(List(0, 1),42)
```

__Nota bene:__ Here, for sake of simplicity, I did not implement `combineAll` with `foldLeft` so I don't have to explain `foldLeft`, but you should know that my `accumulate` inner function __is__ `foldLeft` and that `combineAll` should in fact be implemented like that:

```scala
def combineAll[A](as: List[A])(implicit M: Monoid[A]): A = as.foldLeft(M.id)(M.combine)
```

Voilà !

# More material

If you want to keep diving deeper, some interesting stuff can be found on my [FP resources list](https://github.com/mmenestret/fp-ressources) and in particular:

- [Scala with Cats - Semigroup and monoid chapters](https://underscore.io/books/scala-with-cats/)
- [Why Spark can’t foldLeft: Monoids and Associativity](https://parkergordon.io/2017/04/03/why-spark-cant-foldleft/)
- [Cats documentation](https://typelevel.org/cats/typeclasses/semigroup.html)
- Let me know if you need more

# Conclusion

To sum up, we saw:

- How simple __semigroups__ and __monoids__ are and how closely related they are
- We saw examples of _semigroups_ and _monoids_
- We had an insight about how useful these FP constructs can be in real life
- And finally we showed how they are encoded in _Scala_

I'll try to keep that blog post updated.
If there are any additions, imprecision or mistakes that I should correct or if you need more explanations, feel free to contact me on Twitter or by mail !
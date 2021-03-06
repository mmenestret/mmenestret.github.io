---
layout: post
comments: true
title: "Anatomy of functors and category theory"
author: "Myself"
date: 2019-02-16
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
- Part 5: [Anatomy of functors and category theory]({{ site.baseurl }}{% post_url 2019-02-16-functors-cat-theory %})
- Part 6: Anatomy of the tagless final encoding - Yet to come !

# Introduction

In this article, I'll try to give you an intuition about what is a _functor_ and what do they look like in _Scala_. Then the ones being curious about theory can keep on reading because we'll take a quick glance at category theory and what are _functors_ in category theory terms. Then we'll try to bridge the gap between category theory and pure FP in _Scala_ and finally take a look back at our _functors_ !  

# What is a _functor_ ?

There's a nice answer [by Bartosz Milewski on _Quora_](https://www.quora.com/Functional-Programming-What-is-a-functor) from which I'll keep some parts:

> I like to think of a _functor_ as a generalization of a container. A regular container contains zero or more values of some type. A _functor_ may or may not contain a value or values of some type (...) .
>
> So what can you do with such a container? You might think that, at the minimum, you should be able to retrieve values. But each container has its own interface for accessing values. If you try to specify that interface, you're Balkanizing containers. You're splitting them into stacks, queues, smart pointers,  futures, etc. So value retrieval is too specific.
>
> It turns out that the most general way of interacting with a container is by modifying its contents using a function.

## Let's try to rephrase that

- _Functors_ represent containers
- For now, we won't care about their particularities, all we need to know is that, at some point, they will maybe hold a value or values "inside" (but keep in mind that every container have particularities, I'll refer to that at the end)
- Defining an generic interface about how to access values inside a container does not make any sense since some containers' values would be accessed by index (_arrays_ for example), others only by taking the first element (_stacks_ for example), other by taking the value only if it exists (_optionals_), etc.
- __However__, we can define an interface defining how the value(s) inside containers is modified by a function despite being in a container

__So, to summarize, a _functor_ is a kind of container that can be mapped over by a function.__

But _functors_ have to respect some rules, called _functor_'s laws...

- __Identity__: A _functor_ mapped over by the identity function (the function returning its parameter unchanged) is the same as the original _functor_ (the container and its content remain unchanged)
- __Composition__: A _functor_ mapped over the composition of two functions is the same as the _functor_ mapped over the first function and then mapped over the second one  

> A quick note about _functor_ / _container_ parallel: the analogy is convenient to get the intuition, but not all _functors_ will not fit into that model, keep it in a corner of you mind so that you're not taken off guard.

# How does it look like in practice

Along the next sections, the examples and code snippets I'll provide will be in _Scala_. 

## Let explore some examples

We're going to play with concrete containers of `Int` values to try to grasp the concept.

```scala
val halve: Int => Float = x => x.toFloat / 2
```

Here we defined the function from `Int` to `Float` that we are going to use to map over our containers

- Our first guinea pig is `Option[Int]`, which is a container of (0 or 1) `Int`.

    ```scala
    val intOpt: Option[Int] = Some(99)
    val mappedResult1: Option[Float] = intOpt.map(halve)
    ```

    We can see that an `Option[Int]` turns into an `Option[Float]`, the inner value of the container being modified from `Int` to `Float` when mapped over with a function from `Int` to `Float`... 

- Our second guinea pig is `List[Int]`, which is a container of (0 or more) `Int`.

    ```scala
    val intList: List[Int] = List(1, 2, 3)
    val mappedResult2: List[Float] = intList.map(halve)
    ```

    We can see that an `List[Int]` turns into a `List[Float]`, the inner values of the container are modified from `Int` to `Float` when mapped over with a function from `Int` to `Float`... 

- Our third is a hand made `UselessContainer[Int]`, which is a container of exactly 1 `Int`.

    ```scala
    final case class UselessContainer[A](innerValue: A)
    val intContainer: UselessContainer[Int] = UselessContainer(99)
    val mappedResult3: UselessContainer[Float] = intContainer.map(halve)
    ```

    We can see that an `UselessContainer[Int]` turns into an `UselessContainer[Float]`, the inner value of the container being modified from `Int` to `Float` when mapped over with a function from `Int` to `Float`... (I've deliberately hidden an implementation detail here for clarity, I'll cover it later)


So we can observe that pattern we described earlier: 

__A _functor_, let's call it `F[A]`, is a structure containing a value of type `A` and which can be mapped over by a function of type `A => B`, getting back a _functor_ `F[B]` containing a value of type `B`.__


## How do we abstract and encode that ability ?

_Functors_ are usually represented by a _type class_.

As a reminder, a _type class_ is a group of types that all provide the same abilities (interface), which make them part of the same class (group, "club") of same abilities providing types (see my article about _type classes_ [here]({{ site.baseurl }}{% post_url 2018-10-05-typeclasses %})).

This is the _functor type class_ implementation:

```scala
trait Functor[F[_]]{
    def map[A, B](fa: F[A], func: A => B): F[B]
}
```

1. The types our _functor_ _type class_ abstract over are _type constructors_ (`F[_]`, our container types)
2. The _type class_ exposes a `map` function taking a container `F[A]` of values of type `A`, a function of type `A => B` and return a `F[B]`, a container of values of type `B`: the pattern we just described.

> **A note about _type constructors_**: A _type constructor_ is a _type_ to which you have to supply an other _type_ to get back a new _type_. You can think of it just as functions that take values to produce values. And that makes sense, since we have to supply to our container type the type of values it will "hold" ! 
>
> Most used concrete _type constructors_ are `List[_]`, `Option[_]`, `Either[_,_]`, `Map[_, _]` and so on. 

To illustrate what it means in your _Scala_ code let's make our `UselessContainer` a _functor_:

```scala
implicit val ucFunctor = new Functor[UselessContainer] {
    override def map[A, B](fa: UselessContainer[A],
                           func: A => B): UselessContainer[B] =
      UselessContainer(func(fa.innerValue))
}
```

Be careful, if you attempt to create your own _functor_, it is not enough. __You have to prove that your _functor_ instance respects the _functor_'s laws we stated earlier__ (usually via property based tests), hence that:

- For all values `uc` of type `UselessContainer`: 
    ```scala
    ucFunctor.map(uc, identity) == uc
    ```
- For all values `uc` of type `UselessContainer` and for any two functions `f` of type `A => B` and `g` of type `B => C`:
    ```scala
    ucFunctor.map(uc, g compose f) == ucFunctor.map(ucFunctor.map(uc, f), g)
    ```        

However, you can safely use _functor_ instances brought to you by _Cats_ or _Scalaz_ because their implementations __lawfulness__ are tested for you.

(You can find the _Cats_ _functor_ laws [here](https://github.com/typelevel/cats/blob/master/laws/src/main/scala/cats/laws/FunctorLaws.scala) and their tests [here](https://github.com/typelevel/cats/blob/master/laws/src/main/scala/cats/laws/discipline/FunctorTests.scala). They are tested with [discipline](https://typelevel.org/cats/typeclasses/lawtesting.html).)

Now that you know what a _functor_ is and how it's implemented in Scala, let's talk a bit about category theory !

# An insight about the theory behind _functors_

During this article, we only talked about the most widely known kind of _functors_, the _co-variant endofunctors_. Don't mind the complicated name, they are all you need to know to begin having fun in functional programming.

However if you'd like to have a grasp a little bit of theory behind _functors_, keep on reading.

___Functors_ are structure-preserving mappings between categories.__

## Tiny crash course into category theory

Category theory is the mathematical field that study how things relate to each others in general and how their relations compose.

A category is composed of:

- __Objects__ (view it as something purely abstract, absolutely anything, points for example)
- __Arrows__ or __morphisms__ (which are the ways to go from one object to another)
- And two fundamental properties:
    - __Composition__: A way to compose these arrows associatively. It means that if it exists an arrow from an object `a` to an object `b` and an arrow from the object `b` to an object `c`, it exists an arrow that goes from `a` to `c` and the order of composition does not matter (given 3 morphisms that are composable `f`, `g`, `h` then (`h` . `g`) . `f`) == `h` . (`g` . `f`))
    - __Identity__: There is an identity arrow for every object in the category which is the arrow which goes from that object to itself

![category](https://upload.wikimedia.org/wikipedia/commons/thumb/f/ff/Category_SVG.svg/1024px-Category_SVG.svg.png)

- `A`, `B`, `C` are this category's __objects__
- `f` and `g` are its __arrows__ or __morphisms__
- `g . f` is `f` and `g` composition since `f` goes from `A` to `B` and `g` goes from `B` to `C` (and it __MUST__ exist to satisfy composition law, since `f` and `g` exist)
- `1A`, `1B` and `1C` are the identity arrows of `A`, `B` and `C`

## Back to _Scala_

In the context of purely functional programming in _Scala_, we can consider that we work in a particular category that we are going to call it `S` (I won't go into theoretical compromises implied by that parallel, but there are some !):

- `S` __objects__ are _Scala_'s __types__
- `S` __morphisms__ are _Scala_'s __functions__
    - __Composition__ between morphisms is then __function composition__
    - __Identity__ morphisms for `S` objects is __the identity function__
    
Indeed, if we consider the object `a` (the type `A`) and the object `b` (the type `B`), _Scala_ functions `A => B` are morphisms between `a` and `b`.

Given our morphism from `a` to `b`, if it exists an object `c` (the type `C`) and a morphism between `b` and `c` exists (a function `B => C`):

- Then it must exist a morphism from `a` to `c` which is the composition of the two. And it does ! It is (pseudo code):
    - For `g: B => C` and `f: A => B`, `g compose f` 
- And that composition is associative:
    - `(h compose g) compose f` is the same as `h compose (g compose f)`

Moreover for every object (every type) it exists an identity morphism, the identity function, which is the type parametric function: 

- `def id[A](a: A) = a`

We can now grasp how category theory and purely functional programming can relate !

## And then back to our _functors_

Now that you know what a category is, and that you know about the category `S` we work in when doing functional programming in _Scala_, re-think about it. 

A _functor_ `F` being a structure-preserving mapping between two categories means that it maps objects from category `A` to objects from category `F(A)` (the category which `A` is mapped to by the _functor_ `F`) and morphisms from `A` to morphisms of `F(A)` while preserving their relations.

Since we always work with types and with functions between types in Scala, a _functor_ in that context is a mapping from and to the __same category__, between `S` and `S`, and that particular kind of _functor_ is called an __endofunctor__. 

Let's explore how `Option` behaves (but we could have replaced `Option` by any _functor_ `F`):

__Objects__

| Objects in `S` (types) | Objects in `F(S)` |
| -------------          | ----------------- |
| `A`                    | `Option[A]`       |
| `Int`                  | `Option[Int]`     |
| `String`               | `Option[String]`  |

So `Option` type construtor maps objects (types) in `S` to other objects (types) in `S`.

__Morphisms__

Let's use our previously defined:

-  `def map[A, B](fa: F[A], func: A => B): F[B]`. 

If we partially apply `map` with a function `f` of type `A => B` like so (pseudo-code): `map(_, f)`, then we are left with a new function of type `F[A] => F[B]`. 

Using `map` that way, let's see how morphisms behave:


| Morphisms in `S` (function between types) | Morphisms in `F(S)`               |
| -------------                             | -----------------                 |
| `A => A` (identity)                       | `Option[A] => Option[A]`          |
| `A => B`                                  | `Option[A] => Option[B]`          |
| `Int => Float`                            | `Option[Int] => Option[Float]`    |
| `String => String`                        | `Option[String] => Option[String]`|

So `Option`'s `map` maps morphisms (functions from type to type) in `S` to other morphisms (functions from type to type) in `S`.

We won't go into details but we could have shown how `Option` _functor_ respects morphism composition and identity laws.

## What does it buy us ?

- _Functors_ are mappings between two categories
- A _functor_, due to its theorical nature, preserves the _morphisms_ and their relations between the two categories it maps
- When programming in pure FP, we are in `S`, the category of _Scala_ types, functions and under function composition. The _functors_ we use are then _endofunctors_ (from `S` to `S`) because they map _Scala_ types and functions between them to other _Scala_ types and functions between them

In programming terms, _(endo)functors_ in _Scala_ allow us to move from origin types (`A`, `B`, ...), to new target types (`F[A]`, `F[B]`, ...) while safely allowing us to re-use the origin functions and their compositions on the target types.

To continue with our `Option` example, `Option` type constructor "map" our types `A` and `B` into `Option[A]` and `Option[B]` types while allowing us to re-use functions of type `A => B` thanks to `Options`' `map`, turning them into `Option[A] => Option[B]` and preserving their compositions. 

But that is not over ! Let's leave abstraction world we all love so much and head back to concrete world.

Concrete _functors_  instances enhance our origin types with new capacities. __Indeed, _functor_ instances are concrete data structures__ with particularities (the one we said we did not care about at the beginning of that article), the abilty to represent empty value for `Option`, the ability to suspend an effectful computation for `IO`, the ability to hold multiple values for `List` and so on !

Ok, so, to sum up, why _functors_ are awesome ? The two main reasons I can think of are:

1. Abstraction, abstraction, abstraction... Code using _functors_ allows you to only care about the fact that what you manipulate is mappable.
    - It increases code reuse since a piece of code using _functors_ can be called with any concrete _functor_ instance 
    - And it reduces a lot error risks since you _have to_ deal with less particularities of the concrete, final, data structure your functions will be called with
2. They add functionnalities to existing types, while allowing to still use functions on them (you would not want to re-write them for every _functor_ instances), and that's a big deal:
    - `Option` allow you to bring `null` into a concrete value, making code a lot healthier (and functions purer)
    - `Either` allow you to bring computation errors into concrete values, making dealing with computation errors a lot healthier (and functions purer)
    - `IO` allow you to turn a computations into a values, allowing better compositionality and referential transparency
    - And so on...

I hope I made a bit clearer what _functors_ are in the context of category theory and how that translates to pure FP in _Scala_ !

# More material

If you want to keep diving deeper, some interesting stuff can be found on my [FP resources list](https://github.com/mmenestret/fp-ressources) and in particular:

- [Scala with Cats - Functor chapter](https://underscore.io/books/scala-with-cats/)
- [Functors' section of the awesome Functors, Applicatives, And Monads In Pictures article](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html#functors)
- [Yann Esposito's great "Category theory and programming"](http://yogsototh.github.io/Category-Theory-Presentation/categories.html)
- Let me know if you need more

# Conclusion

To sum up, we saw:

- That a _functor_ is a kind of container that can be mapped over by a function and the laws it has to respect
- Some examples and identified a common pattern
- How we abstract over and encode that pattern in _Scala_ as a _type class_ of _type constructors_
- We had a modest overview about category theory, what _functors_ are in category theory, and how both relates to pure FP in _Scala_
- We concluded by how great _functors_ are and for what practical reasons

I'll try to keep that blog post updated.
If there are any additions, imprecision or mistakes that I should correct or if you need more explanations, feel free to contact me on Twitter or by mail !

---

___Edit__: Thanks [Jules Ivanic](https://twitter.com/guizmaii) for the review :)._
---
layout: post
title: "Anatomy of a type class"
author: "Martin Menestret"
date: 2018-10-05
tags: [Scala, Functional programming]
---

I will try to group here, in an anatomy atlas, basic notions of functional programming that I find myself explaining often lately into a series of articles.

The idea here is to have a place to point people needing explanations and to increase my own understanding of these subjects by trying to explain them the best I can.
I'll try to focus more on making the reader feel an intuition, a feeling about the concepts rather than on the perfect, strict correctness of my explanations.

- Part 1: [Anatomy of a type class]({{ site.baseurl }}{% link _posts/2018-10-05-typeclasses.md %})
- Part 2: Anatomy of an algebraïc data types (ADT) - Not written yet
- Part 3: Anatomy of functors, applicatives and monads - Not written yet

# Motivation

## Data/behavior relationship

_OOP_ and _FP_ have two different approaches when it comes to data/behavior relationship:

- _Object oriented programming_ often __combines data and behavior__ by mixing them into classes that:
    - Store data as an internal state
    - Expose methods that act on it and may mutate it

```scala
case class Player(nickname: String, var level: Int) {
    def levelUp(): Unit          = { level = level + 1 }
    def sayHi(): String          = s"Hi, I'm player $nickname, I'm lvl $level !"
}
```

- _Functional programming_ aims to completely __separate data from behavior__ by:
    - Defining types (ADT) on one side that expose no behavior and only holds data
    - Functions taking values of some of these types as inputs, acting on them and outputting values of some of those types (leaving the input values unchanged)

```scala
case class Player(nickname: String, var level: Int)

object PlayerOperations {
    def levelUp(p: Player): Player = p.copy(level = p.level + 1)
    def sayHi(p: Player): String   = s"Hi, I'm player ${p.nickname}, I'm lvl ${p.level} !"
}
```

## Polymorphism

_Ad hoc polymorphism_ is defined on _Wikipedia_ by:

> Ad hoc polymorphism is a kind of polymorphism in which polymorphic functions can be applied to arguments of different types, because a polymorphic function can denote a number of distinct and potentially heterogeneous implementations depending on the type of argument(s) to which it is applied

It is a mechanism allowing a function to be defined in such a way that the actual _function implementation_ that is going to be called at runtime depends on the parameter types it is called with.

You won't have to define `printInt`, `printDouble`, `printString` functions, a `print` function might be enough.

The purpose of that is mainly to program by manipulating _interfaces_ (as a concept, the layer allowing elements to communicate, not as in a _Java Interface_ even their purpose is to encode that concept) exposing shared, common, behaviors and use these behaviors instead of writing different function implementations for each of the concrete types abstracted by these interfaces.
Your `print` function might somehow require a `Printable` interface, abstracting for its argument the ability to print themselves.

- _Object oriented programming_ often use __subtyping via interface inheritance__ to permit polymorphism by making concrete classes inherit interfaces exposing the needed shared behaviors
- _Functionnal programming_, willing to strongly separate data and behavior __favours type classes__ which allows to add functionalities to existing types without having to modify them or to know they will need these functionnalities beforehand.

# What's a type class ?

A _type class_ can be described literally as a class of types, a grouping of types, that shares common capabilities.

It represents an abstraction of something that a grouping of types would have in common, just as _"Things that can say Hi"_ abstracts over every concrete types that have the ability to greet or as _"Things that have petals"_ might abstract over flowers in the real world.

# How can it be done in Scala ?

_Type classes_ are not specific to _Scala_ and can be found in many functional programming languages.
They are not a first class construct in _Scala_, as it can be in _Haskell_ but it still can be done quit easily.

In _Scala_ it is encoded by a _trait_ which exposes the _"contract"_ of the _type class_, what the _type class_ is going to abstract over, and by the _concrete implementations_ of that trait for every types that we want to be instances of that _type classes_.

Our _type class_ for _"types that can say Hi"_ is going to be:

```scala
trait CanGreet[T] {
    def sayHi(t: T): String
}
```

Representing all the types `T` which have the capability to `sayHi`.

Given a:

```scala
case class Player(nickname: String, level: Int)
val geekocephale = Player("Geekocephale", 42)
 ```

 We now create a `Player` instance of the `CanGreet` _trait_ for it to be an instance of our _type class_:

 ```scala
val playerGreeter = new CanGreet[Player] {
    def sayHi(t: Player): String = s"Hi, I'm player ${t.nickname}, I'm lvl ${t.level} !"
}
```

Thanks to what we did, we can now define generic functions such as:

```scala
def greet[T](t: T, greeter: CanGreet[T]): String = greeter.sayHi(t)
```

`greet` is polymorphic in the sense that it will work on any `T` as long as it gets an instance of `CanGreet` for that type.

```scala
greet(geekocephale, playerGreeter)
```

However, that is a bit cumbersome to use, and we can leverage _Scala_'s _implicits_ power to make our life easier (and to get closer to what's done in other languages' _type class_ machinery).
Let's redifine our `greet` function, and make our `CanGreet` instance _implicit_ as well:

```scala
def greet[T](t: T)(implicit greeter: CanGreet[T]): String = greeter.sayHi(t)

implicit val playerGreeter = new CanGreet[Player] {
    def sayHi(t: Player): String = s"Hi, I'm player ${t.nickname}, I'm lvl ${t.level} !"
}
```

Now, we can call our `greet` function without explicitly passing the `CanGreet` instance as long as we have an _implicit_ instance for the type `T` we are using in scope (which we have, `playerGreeter`) !

```scala
greet(geekocephale)
```

# Optionnal cosmetics

That part is absolutely not mandatory to understand how _type classes_ work, it is just about common syntax additions to what we saw before, mostly for convenience, and it can be skipped to go directly to conclusion.

```scala
def greet[T](t: T)(implicit greeter: CanGreet[T]): String
```

Is strictly the same thing and can be refactored as:

```scala
def greet[T: CanGreet](t: T): String
```

The function signature looks nicer, and I think it expresses better the _"need"_ for `T` to _"be"_ an instance of `CanGreet`, but there is a drawback: we lost the possibility to refer to our `CanGreet` implicit instance by a name.
To do so, in order to summon our instance from the implicit scope, we can use the `implicitly` function:

```scala
def greet[T: CanGreet](t: T): String = {
    val greeter: CanGreet[T] = implicitly[CanGreet[T]]
    greeter.sayHi(t)
}
```

To make it less cumbersome, you'll commonly see a companion object for _type classes_ traits with an `apply` method:

```scala
object CanGreet {
    def apply[T](implicit C: CanGreet[T]): CanGreet[T] = C
}
```

It does exactly what we did in our last `greet` function implementation, allowing us to now re-write our `greet` function as follows:

```scala
def greet[T: CanGreet](t: T): String = CanGreet[T].sayHi(t)
```

`CanGreet[T]` is calling the companion object `apply` function (`CanGreet[T]` is in fact desugarized as ``CanGreet.apply[T]()`) to summon `T`'s `CanGreet` instance from implicit scope and we can immediately use it in our `greet` function by calling `.sayHi(t)` on it.

Finally, you'll also probably see _implicit classes_, called _syntax_ for our _type class_ that holds the operation our _type class_ permits:

```scala
implicit class CanGreetSyntax[T: CanGreet](t: T) {
    def greet: String = CanGreet[T].sayHi(t)
}
```

Allowing our `greet` function to be called in a more convenient, OOP method way:

```scala
geekocephale.greet
```

# Tooling

- [Simulacrum](https://github.com/mpilquist/simulacrum) is an awesome library that helps tremendously reducing the boilerplate by using annotations that will generate the _type classe_ and syntax stuff for you at compile time
- [Magnolia](https://github.com/propensive/magnolia) is a great library as well to automaticly derive your _type classes_ instances

We did not talk about _type classes_ derivation which is a bit more advanced topic, but the basic idea being that, if your types `A` and `B` are instances of a _type class_, and if you have a type C formed by combining `A` and `B`, such as:

```scala
case class C(a: A, b: B)
```

or

```scala
sealed trait C
case class A() extends C
case class B() extends C
```

It makes the type `C` automatically an instance of your _type class_ !

# More material

If you want to keep diving deeper, some interesting stuff can be found on my [FP resources list](https://github.com/mmenestret/fp-ressources) and in particular:

- [Mastering Typeclass Induction](https://www.youtube.com/watch?v=Nm4OIhjjA2o)
- [Type classes in Scala](https://blog.scalac.io/2017/04/19/typeclasses-in-scala.html)
- [Implicits, type classes, and extension methods](https://kubuszok.com/compiled/implicits-type-classes-and-extension-methods/)

# Conclusion

So to conclude here, we saw why _type classes_ are useful, what they are, how they are encoded in _Scala_ and some cosmetics and tooling that might help you to work with them.

We went through:

- How to create a _class_ of types that provides shared behaviors

    - Without __having to know it beforehand__ (we don't extend our types nor modify them)
    - Without __having to mix that behavior to the data__ (the case class itself)

- How to create new instances of our _type class_
- How to leverage polymorphism by designing __polymorphic functions__ defined and working for any type `T` as long as an (implicit) instance of it's required _type class_ is provided !

I'll try to keep that blog post updated.
If there are any additions, imprecision, or mistakes that I should correct or if you need more explanations, feel free to contact me on Twitter or by mail !

# Functional programming constructs anatomy

I will try to group here basic notions of functional programming that I find myself explaining a lot lately into a series of articles.
The idea here is to have a place to point people needing explanations and to increase my own understanding by trying to explain them the best I can.

Part 1: Type classes
Part 2: Algebraïc data types (ADT) - Not written yet
Part 3: Functor, applicatives and Monads - Not written yet

## Type classes

### Motivation

#### Data / behavior relationship

OOP and FP have two different approaches about data / behavior relationship:

- Object oriented programing often combines data and behavior by mixing them into classes that:
    - Store data as an internal state
    - Expose methods that act on it

- Functional programming aims to completly separate data from behavior as two separate things by:
    - Defining data types (ADT) on one side that expose no behaviors at all and only holds data
    - Functions taking values of some of these types as inputs, acting on them and outputing values of some of those types (leaving the input values unchanged)

#### Polymorphism

Ad hoc polymorphism is a class of polymorphism, defined that way by Wikipedia:

> Ad hoc polymorphism: defines a common interface for an arbitrary set of individually specified types

__Important note__: _interface_ here is meant as a way to expose behaviors shared by sevral possibly unrelated types.

The purpose of that is mainly to program by manipulating interfaces exposing common behaviors and act on them based on these behaviors instead of writing different functions for each of the different concrete types abstracted by these interfaces.

An simple example would be to define a _Serializable_ interface to represents the ability for types to be serialized.
A function that would send a serialized message to a queue would only need to know that it acts on a _Serializable_ interface to be implemented and then only one implementation would be enough for all types abtracted by this interface.

- Object oriented programming usualy uses interface inheritance to permit polymorphism by making concrete classes inherit interfaces exposing the needed shared behaviors
- Functionnal programming, willing to strongly separate data and behavior favours type classes which allows to add functionnalities to existing types without having to modify them or to know they will need these functionnalities beforehand.

### What's a type class ?

A type class can be described literraly as a class of types, a grouping of type, that shares common capabilities.

Type classes are not specific to _Scala_ and can be found in many functional programming languages.

In Scala it is encoded by a trait which exposes the "contract" of the type class, and by the concrete implementations of that trait for every single types that needs to be instances of that type classes.

Our type class is going to be:

```scala
trait CanGreet[T] {
    def sayHi(t: T): String
  }
```

Representing all the types `T` which should have the capability to `sayHi`

Given a:

```scala
case class Player(nickname: String, level: Int)
val geekocephale = Player("Geekocephale", 42)
 ```

 We now create an instance of CanGreet for our Player to be a member of that type class:

 ```scala
val playerGreeter = new CanGreet[Player] {
    def sayHi(t: Player): String = s"Hi, I'm player ${t.nickname}, I'm lvl ${t.level} !"
}
```

Thanks to what we did, we can now define a generic function:

```scala
def greet[T](t: T, greeter: CanGreet[T]): String = greeter.sayHi(t)
```

`greet` is polymorphic in the sense that it will work on any time `T` if it gets an instance of `CanGreet` for that type type.

```scala
greet(geekocephale, playerGreeter)
```

However, that is a bit cumbersome and we can leverage implicits power to make our life easier.
Let's redifine our greet function, and make our `CanGreet` instance _implicit_:

```scala
implicit val playerGreeter = new CanGreet[Player] {
    def sayHi(t: Player): String = s"Hi, I'm player ${t.nickname}, I'm lvl ${t.level} !"
}
def greet[T](t: T)(implicit greeter: CanGreet[T]): String = greeter.sayHi(t)
```

Now, we can call our `greet` function that way, and it will compile and work as long as it has a `CanGreet` instance for our `T` in its _implicit scope_ (which it does, `playerGreeter`).

```scala
greet(geekocephale)
```

So to conclude here, we saw how we could, thanks to type classes

- Design a __polymorphic function__ in the sense that it works for any type `T` that provides an (implicit) instance of `CanGreet`
- Create a _class_ of types that provides a shared behavior
    - Without __having to know it beforehand__
    - Without __mixing that behavior to the data (the case class itself)__
- How to add new instances to our type class

### Optionnal cosmetics

That part is absolutly not mandatory to understand how type classes work, it is just about common syntax additions to what we saw before, mostly for convenience, and it can be skipped to go directly to conclusion.

```scala
def greet[T](t: T)(implicit greeter: CanGreet[T]): String
```

Is strictly the same thing and can be refactored as:

```scala
def greet[T: CanGreet](t: T): String
```

The function signature looks nicer, and I feel like it expresses better the _"need"_ for `T` to _"be"_ an instance of `CanGreet`, but there is a drawback: we lost the possibility to refer to our CanGreet implicit instance by its name.
To do so, in order to summon our instance from the implicit scope, we can use the `implicitly` function:

```scala
def greet[T: CanGreet](t: T): String = {
    val greeter: CanGreet[T] = implicitly[CanGreet[T]]
    greeter.sayHi(t)
}
```

You'll commonly see an companion object for type classes traits with an apply method similar to:

```scala
object CanGreet {
    def apply[T](implicit C: CanGreet[T]): CanGreet[T] = C
}
```

It does exactly what we did in our `greet` function, and it allow us to now re-write our greet function as follows:

```scala
def greet[T: CanGreet](t: T): String = CanGreet[T].sayHi(t)
```

`CanGreet[T]` is calling the companion object apply method to summon `T`'s `CanGreet` instance from implicit scope.

And finaly, you'll also probably see _implicit classes_, called _syntax_ for our type class:

```scala
implicit class CanGreetSyntax[T: CanGreet](t: T) {
    def greet: String = {
      CanGreet[T].sayHi(t)
    }
}
```

Allowing our greet method to be called in a more convenient, method like, way:

```scala
geekocephale.greet
```

### Conclusion

So to conclude here, we saw why we need typeclasses, what they are, and how we could:

- Create a _class_ of types that provides a shared behavior
    - Without __having to know it beforehand__
    - Without __mixing that behavior to the data (the case class itself)__
- How to add new instances to our type class
- Leverage polymorphism by designing a __polymorphic function__ defined and working for any type `T` that provides an (implicit) instance of it's required typeclass !



I'll try to keep that blog post updated, feel free to contact me for any additions, corrections or questions :).
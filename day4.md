# ??? Advanced Scala Day 4

## Stateful Representations

Stream  -------------- compile --------> IR
- reified description                    - reified description


## Group Theory

A group is a concept from a branch of mathematics called abstract algebra.

A set A has a group if we can define:

- a binary operation + with type (A, A) => A;
- an identity element 0 in A, s.t. 0 + a = a = a + 0;
- for every element a in A, there is an element -a in A s.t. a + -a = 0 = -a + a; and
- + is associative, meaning (a + b) + c = a + (b + c)

Examples:
- rotations
- translations
- rubik's cube


### Why Do We Care?

We have the usual benefit of abstraction. When we define an abstraction:

- we can make our code generic over any type for which we can define that abstraction; and
- we can use that abstraction in our thinking.

We can also do maths, but that is less of a goal for most programmers.

Group probably doesn't seem like a very useful abstraction in code. A lot of theoretical CS uses group theoretical ideas, but let's look at two abstractions that are directly useful in data engineering: semigroup and monoid.


## Semigroup

A set A has a semigroup if we can define:

- a binary operation + with type (A, A) => A; and
- + is associative, meaning (a + b) + c = a + (b + c)

(You can see that a semigroup is a group with some restrictions removed.)


## Monoid

A set A has a monoid if we can define:

- a binary operation + with type (A, A) => A;
- an element 0 in A, s.t. 0 + a = a = a + 0; and
- + is associative, meaning (a + b) + c = a + (b + c)

A monoid is a semigroup with an identity.


## Monoids in Data Engineering

In many cases the operations we do in data science can be reduced to monoids (or, at least, a monoid forms part of it.) For example, if we're collecting the set of unique users who accessed a web site, set union is a monoid. If we're adding up the total users, that's also a monoid. The associative property means that parallel processing is also fine.

```
(a + b) + c == a + (b + c)
```

and imagine that the brackets indicate which machine is computing the partial result.

If the monoid is commutative, meaning `a + b == b + a`, then parallel processing in any order gets the correct answer.

https://arxiv.org/abs/1304.7544


## Implementation

The obvious OO implementation might be something like

```scala
trait Semigroup[A] {
  def combine(that: A): A
}

class Int extends Semigroup[Int] {
  def combine(that: Int): Int =
    this + that
}
```

I.e. define an interface and extend the interface for any type that has, e.g., a semigroup.

Problems:

- How do we go about modifying code that already exists, some of which is part of the JVM (e.g. `Int`)?

- Some types have, for example, multiple monoids. E.g. for `Int` there is (+, 0) and (*, 1).


## Type classes

Separate implementation of the interface from the type that we're implementing it for.

E.g. implementation of `Semigroup` for `Int` is separate from the implementation of `Int`.

Add some languages features (implicits) to make this easier to work with.

Without implicits:

```scala
// Define a type class
trait Semigroup[A] {
  def combine(x: A, y: A): A
}

// Implement the type class for a given type
// Called a type class instance
object IntSemigroup extends Semigroup[Int] {
  def combine(x: Int, y: Int): Int =
    x + y
}

// Use the type class
object SemigroupExample {
  def combine[A](x: A, y: A)(semigroup: Semigroup[A]) =
    semigroup.combine(x, y)
    
  combine(1, 2)(IntSemigroup)
}
```

This is inconvenient. Passing the type class instances around is annoying, and combining (composing) type class instances to produce new ones is super annoying.

Type class composition:

```scala
// We can compose type classe instances. For example, construct an instance for Option[A] if an instance for A exists.
object OptionSemigroup {
  def apply[A](semigroup: Semigroup[A]): Semigroup[Option[A]] =
    new Semigroup[Option[A]] {
      def combine(x: Option[A], y: Option[A]): Option[A] =
        x match {
          case None => y
          case Some(x1) =>
            y match {
              case None = Some(x1)
              case Some(y1) => Some(semigroup.combine(x1, y1))
            }
        }
    }
}

// The composition OptionSemigroup(IntSemigroup) would quickly get very tedious to construct
SemigroupExample.combine(Option(1), Option(2))(OptionSemigroup(IntSemigroup))
```


## Implicits

Scala's implicits solve these issues:

- compiler will pass type class instances to methods that expect them
- compiler will construct instances via composition when we want it to

Three (+1) things under the umbrella "implicit" in Scala 2:
- implicit parameters
- implicit values
- implicit classes
- implicit conversions, which are evil and we won't talk about further


### Implicit Values

An implicit value is:

1. `implicit val`
2. `implicit object`
3. `implicit def` with only implicit parameters.

val / object / def defined in this way acts like a normal val / object / def but it also becomes an implicit value, which means the compiler can pass it to an implicit parameter.


### Implicit Parameters

Implicit parameter lists are:

- the last parameter list of a method
- starts with the keyword `implicit`
- applies to all parameters in that list (all the parameters in the list become implicit parameters)

If we don't explicitly pass an implicit parameter, the compiler looks for an implicit value of the correct type. If it finds one it will pass that value for the parameter.

```scala
implicit val name: String = "Noel"

def hello(n: Int)(implicit subject: String) =
  s"Hello $subject" * n


hello(1) // "Hello Noel"
```


### Implicit Scope and Ambiguity

If the compiler finds multiple implicit values that match the type of the implicit parameter, it will give an "ambiguous implicit value" error. It doesn't choose a value.

What does the compiler look for implicit values? This is called the *implicit scope* and it includes:

- the normal (lexical) scope where values are found. (imports and enclosing {})
- the companion object of any type involved in the lookup

Types involved:

- any type in the type of the parameter (e.g. if the parameter has type `Option[Int]` it would look at the companion objects for `Option` and `Int`)
- also companion object, if applicable, of the type on which we are calling the method

=> By convention, people put type class instances (implicit values) in companion objects because the compiler automatically looks there (so no need for the user to import anything).


## Secret Type Class Decoder Wheel

- Type class == trait
- Type class instance == implicit value
- Type class usage == implicit parameter

Implicit def with implicit parameter can be used for type class instance composition. I.e. building type class instances from existing instances.

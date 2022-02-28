# ??? Advanced Scala Day 1

## Introductions

Name, programming background, experience with Scala, things you want to learn / issues you've run into that you want to solve.

Noel: 20 yrs Java, Scala, JS, C, Python, R. 10 yrs Scala. ML (RL & Bayesian), PL, teaching. 

Swapnil: Python, Java, some Scala. Build on foundations and learn more about FP techniques. 

Amy: Python, R, SAS (not sass). Data analysis. Building more knowledge of FP approaches.

Gali: Self-taught Python, switched to Scala (learned on job / 2019). Most experience using Python for data science. Aiming to understand particular places where Scala is more useful than Python. Kids are good!

Jun: Fortran (!!!), SAS, Python. How to program in a way that is readable / maintainable / scalable.

Maggie: Python / Java / C. Just started on migrating Scala to Python. Better understand structure / clean code.

Mehul: Java & Python. Get more knowledge of Scala. How to use it on a day-to-day basis. Team is heavily dependent on Scala. 

Dipesh: Expand Scala knowledge, understand every concept.


## Admin

1. Cameras / mics on please, unless you have a good reason
2. Pets (and small children) are welcome to join us, but you should introduce them to everyone else. Older children, friends, housemates, partners, etc. are also welcome.
3. You are ultimately responsible for your learning. Three hints:
   1. Ask questions at any time. "I don't understand this and I don't know how to form a clearer question" is a great question!
   2. Collaborate with other students. Explain what you are doing to them. Explain what you're thinking. "I don't know what to do right now" is a great thing to say.
   3. Take notes.

Goals
1. Overall goal: learn how to design and build systems in Scala
2. Basic principles of functional programming and their application in system design. The importance of semantics.
3. Interpreters and their implementation techniques
4. Type classes in general, and specific type classes that are useful

Admin
- Our curriculum is not fixed. We have a rough plan but it will change depending on your interests.
- We have 5 3.5 hr sessions. We'll have a 30 min break about half way through


## Code Reading

Read the `SizeHint.scala` code in the Scalding project.

https://github.com/twitter/scalding/blob/b0ba993ac817e6b1e52126e8b1cfb1054cc00dad/scalding-core/src/main/scala/com/twitter/scalding/mathematics/SizeHint.scala

Do you understand the semantics of the language constructs used?
Do you understand why they are used?


## Algebraic Data Type

Data defined in terms of logical ands and logical ors

```scala
// A is a B or C
sealed trait A
final case class B() extends A
final case class C() extends A

sealed abstract class A
final case class B() extends A
final case class C() extends A
```

The difference between `sealed trait` and `sealed abstract class` is extremely minor. 

- `sealed abstract class` is what you'll see in the Scala standard library. It has better interop with Java. Might be slightly faster (but would have to benchmark).

- `sealed trait` is what we teach in beginner courses because we have to introduce fewer concepts (we use traits in type classes as well)


```scala
// A is a B and C
final case class A(b: B, c: C)
```

People will sometimes skip out the `final` on a `case class`. This is a modest evil. They almost always intend the case class to be final.


## Structural Recursion

Transform an algebraic data type into anything else

Two implementation methods:
- pattern matching
- dynamic dispatch

### Pattern Matching

Classic FP style

```scala
// A is a B or C
anA match {
  case B() => ???
  case C() => ???
}

// A is a B and C
case A(b, c) => ???
```

### Dynamic Dispatch

Classic OO

```scala
// A is a B or C
sealed abstract class A {
  def someMethod: D  // abstract method---no implementation here
}
// B is a E and F
final case class B(e: E, f: F) extends A {
  def someMethod: D = ??? 
    // Concrete implementation here
    // Can refer to e and f in the body of the method
}
final case class C() extends A {
  def someMethod: D = ??? // Concrete implementation here
}
```
### Recursion Rule

If the data is recursive, the method is recursive at that point.

```scala
sealed abstract class IntList {
  def sum: Int =
    this match {
      case IntPair(h, t) => h + t.sum
      case IntEmpty => 0
    }
}
final case class IntPair(head: Int, tail: IntList) extends IntList
//final case class IntEmpty() extends IntList
case object IntEmpty extends IntList
```


## Language Features

case object
- sometimes an element of an algebraic data type has no data inside it
- in this case we don't need to instantiate instances---they would all be the same
- use a case object instead
- don't need to mark them final as object are not classes and cannot be extended

`@` syntax in pattern matching
- `case someName @ SomePattern(...) =>` this gives the name `someName` to the entire object that is matched, and the name can be referred to on the RHS of the `=>`

```scala
anA match {
  case b @ B() => b.doSomething
}
```

In the above example, assume `anA` has type `A`. However `b` has type `B`, which is more specific than `A`. This means we can call methods defined on `B` but not on `A`.

Class vs object
- an object is a value. It is a thing we can use in our program. Call methods on it. Access instance variables etc.
- a class is a template for creating objects. It is not a value itself. We don't call methods on classes. We create instances of classes (objects) and call methods on those instances.


## Layers

Theory: concepts. Ideas we are implementing and using to think with
------------------------------------------------
Code: craft. Implementation of concepts in Scala


## Following The Types

Look at the output type and input types and see if there are methods to go from one to the other.


## Interpreters

Separation between description and action.

A description describes some computation
- a data structure that describes the computation
- the `Expression` in the `Calculator` example
- called an abstract syntax tree in compilers

An interpreter gives meaning to a description
- carries out the action
- different interpreters can carry out different actions
- `eval` and `print` in `Calculator`

Allows, e.g., optimization of description as in Spark.


```scala
// Your mission is to implement a simple calculator
//
// Your missions:
//
// Mission the First is to implement a data structure to represent arithmetic expressions.
//
// An Expression is:
//
// - A Literal, which has a Double value;
// - An Addition, which has a left and right Expression;
// - A Division, which has a left and right Expression;
// - A Multiplication, which has a left and right Expression; or
// - A Subtraction, which has a left and right Expression
//
//
// Mission the Second is to implement two "interpreters". The first interpreter
// will take an expression and produce a Double using the usual rules of
// mathematics. The second interpreter will take an expression and produce a
// String representing that expression, with correct parentheses to indicate
// precedence. Implement these interpreters in methods that are *not* on the
// Expression type. I.e. create a new object and put them there.
//
// Learning points:
// - Appreciate the utility of algebraic data types and structural recursion. Most
// of the code should follow from the programming strategies.
//
// - Understand how representing the expressions as data ("reifying" them)
// decouples the data representation from the functions that act on it, and
// allows us to implement different interpreters.
object Calculator extends App {
  def eval(expr: Expression): Double =
    expr match {
      case Literal(value)              => value
      case Addition(left, right)       => eval(left) + eval(right)
      case Subtraction(left, right)    => eval(left) - eval(right)
      case Division(left, right)       => eval(left) / eval(right)
      case Multiplication(left, right) => eval(left) * eval(right)
    }

  def print(expr: Expression): String =
    expr match {
      case Literal(value)              => value.toString
      case Addition(left, right)       => s"(${print(left)} + ${print(right)})"
      case Subtraction(left, right)    => s"(${print(left)} - ${print(right)})"
      case Division(left, right)       => s"(${print(left)} / ${print(right)})"
      case Multiplication(left, right) => s"(${print(left)} * ${print(right)})"
    }

  val testExpr = Addition(Literal(1.0), Literal(2.0))
  println(eval(testExpr))
  println(print(testExpr))
}

sealed abstract class Expression
final case class Literal(value: Double) extends Expression
final case class Addition(left: Expression, right: Expression)
    extends Expression
final case class Subtraction(left: Expression, right: Expression)
    extends Expression
final case class Division(left: Expression, right: Expression)
    extends Expression
final case class Multiplication(left: Expression, right: Expression)
    extends Expression
```


## Substitution

Interpreters are very common in FP, to handle effects.
- Cats Effect, Monix, ZIO "effect monads" these are descriptions + interpreters
- Spark
- fs2

1 + 1 + 1
2 + 1
3

x = 1
y = 1 + x
y = 1 + 1
y = 2

Substitution in action!

Substitution is really really easy!

```scala
val x = 1
val y = 1 + x
```

Substitution can be a way to understand programs ("the substitution model of evaluation"). But! Not all code works with substitution.

```scala
val x = println("Hello!")
x
```

is not the same as

```scala
val x = println("Hello!")
println("Hello!")
```

but they would be if substitution held. `println` has a side-effect. This "breaks" substitution.

FP core principle is that being able to understand code before running it (reasoning about code) is really important.
- So can we maintain substitution throughout our code? Because reasoning using substitution is really easy.

We need effects. We need to be able to talk to the outside world! So what do we do?

Separate our program into description and action.
- Descriptions can be freely substituted
- Once we carry out actions we can't do substitution any more
- Carry out actions once at the end of the program (usually).


```scala
final case class Println(output: String)
val x = Println("Hello!")
x

val x = Println("Hello!")
Println("Hello!")
```

This description does not break substitution.

Still need:
- way to build bigger descriptions from smaller ones (composition)
- a way to run these actions.


## Structure of Interpreters

Constructors:
- create descriptions from other values
- type is from something else into description (e.g. `A` to `Stream[A]`)

Combinators
- combine descriptions to produce new descriptions
- type is from a description (often the hidden `this` parameter), and optionally other stuff, to a description

Interpreters
- do stuff; carry out descriptions
- type is from description to something else

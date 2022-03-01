# ??? Advanced Scala Day 2

## Reification

Reification is an implementation technique for interpreters.

Reification:
- abstract meaning is to make concrete something that is abstract
- concrete meaning is to turn method calls into data structures

Capture all the parameters of a method call (including the hidden `this` parameter) and put them into a data structure.

The data structure `extends` the result type of the method call.

Builds a description in the "separation of description from action" model that we're using.

```scala
object Calculator extends App {
  // Interpreters
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
  val testExpr2 = Expression(1.0) + Expression(2.0)
  println(eval(testExpr))
  println(print(testExpr))
}

sealed abstract class Expression {
  // Combinators

  def +(that: Expression): Expression =
    Addition(this, that)
    
  def -(that: Expression): Expression = 
    Subtraction(this, that)
    
  def *(that: Expresssion): Expression =
    Multiplication(this, that)
    
  def /(that: Expression): Expression =
    Division(this, that)
}
object Expression {
  // Constructor
  def apply(value: Double): Expression =
    Literal(value)
}
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


## Reification and Interpreters

Reification is an implementation technique for interpreters.

Reify:
- Constructors. We capture the parameters to the constructor but not `this`
- Combinators. We capture parameters *and* `this`.
- Interpreters we **do not** reify.

Interpreters are then structural recursion over the algebraic data type that we reified into.

This gives us a design technique as well:
- design the methods we want (constructors, combinators, interpreters)
- reify as appropriate
- implement interpreters

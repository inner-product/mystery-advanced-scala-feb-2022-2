# ??? Advanced Scala Day 3

## Plan

- Reification cont...
- Semantics
- State? Probably we'll discuss plans at this point.


## Ideas

Serialization. 

Caching.

Spark semantics, particularly around caching.

```scala
package stream

sealed trait Stream[A] {
  import Stream._

  // Combinators

  /** Keeps all values that match the predicate. */
  def filter(pred: A => Boolean): Stream[A] =
    Filter(this, pred)

  def map[B](f: A => B): Stream[B] =
    Map(this, f)

  // Interpreters

  // Stream.constant(1).map(x => x + 2).filter(x => x < 2).next
  // Upstream: Constant ---> Map ---> Filter ---> Downstream: List
  //               next <--- next <-- next <-----
  // Demand for values (calls to next) go upstream
  // Values go downstream
  // Do we allow one unit of demand (one call to next) to turn into infinite units of demand upstream
  // In the example above, filter will make infinite demand of its upstream as the filter will never be met.
  //
  // Result is
  // - a value; or
  // - no more values ever (stream has ended); or
  // - no more values right now (stream is waiting)
  private def next: Result[A] =
    this match {
      case Constant(value) => Some(value)
      case Emit(values) => values.nextOption()
      case Filter(source, pred) => 
        def loop(): Option[A] =
          source.next match {
            case Some(value) => 
              if (pred(value)) Some(value) else loop()
            case None => None
          }

        loop()
      case Map(source, f) =>
        source.next match {
          case Some(value) => Some(f(value))
          case None => None
        }
    }

  def foldLeft[B](z: B)(f: (B, A) => B): B =
    this.next match {
      case Some(value) => foldLeft(f(z, value))(f)
      case None        => z
    }

  def toList: List[A] =
    foldLeft(List.empty[A])((lst, elt) => elt :: lst).reverse

}
object Stream {
  final case class Constant[A](value: A) extends Stream[A]
  final case class Emit[A](values: Iterator[A]) extends Stream[A]
  final case class Filter[A](source: Stream[A], pred: A => Boolean) extends Stream[A]
  final case class Map[A,B](source: Stream[A], f: A => B) extends Stream[B]

  // Constructors

  /** Creates an infinite Stream that always produces the given value */
  def constant[A](value: A): Stream[A] = Constant(value)
  def emit[A](values: Iterator[A]): Stream[A] = Emit(values)
}
```

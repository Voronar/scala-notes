# Scala notes

## Classes/Traits
- `case classes/objects` for pattern matching
- `sealed base trait` for case classes in order to isolate matching variants (prohibit extending classes outside package scope)
- `values classes` (inherited from AnyVal) for optimisation (raw type representation instead boxed)
- `abstract classes` and `traits` are interchangeable
- `multiple inheritance` problem solves with `linearization` (special Scala algorithm) of applied traits from right most trait to left the left one (`SomeClass with Trait1 with Trait2`)
- `self type` is explicit mark of self type on trait that allow this trait recognise members from another mixed in traits; https://coderwall.com/p/t_rapw/cake-pattern-in-scala-self-type-annotations-explicitly-typed-self-references-explained https://github.com/davidmoten/cake-pattern
- the `.type` singleton type forming operator can be applied to values of all subtypes of Any
  ```scala
  def foo[T](t: T): t.type = t
  foo(23)
  ```

## Companion objects
- `implicit conversions` for companion classes (type classes)
- `extractor` object (with unapply method and optional apply) for custom matching patterns

## Implicits
- `implicit conversion (older style, prefer implicit classes with implicit parameters)` to an expected type via function
converting the receiver; interoperating with new types via conversation to expected class with new methods
  ```scala
  1 + complexStructureWithPlusMethod
  ```
- `implicit classes`; for any such class, the compiler generates an implicit conversion from the class’s constructor parameter to the class itself and makes all methods implicit
- `implicit objects` like classes but lazy; like any object, an implicit object is a singleton but it is marked implicit so that the compiler can find if it is looking for an implicit value of the appropriate type. A typical use case of an implicit object is a concrete, singleton instance of a trait which is used to define a type class
- `context bound syntax`
  ```scala
    def add[A: Monoid](items: List[A]): A =
      items.foldLeft(Monoid[A].empty)(_ |+| _)
    // is sugar for code bellow
    def add[A](items: List[A])(implicit monoid: Monoid[A]): A =
      items.foldLeft(monoid.empty)(_ |+| _)
  ```

## Modularity
- static module = object + trait
- dynamic module = class + trait
- https://github.com/yawaramin/scala-modules
- http://lambdafoo.com/scala-syd-2015-modules
- `package objects` can contain arbitrary definitions, not just variable and method definitions. For instance, they are frequently used to hold package-wide type aliases and implicit conversions. Package objects can even inherit Scala classes and traits.

## Types
- overview https://kubuszok.com/2018/kinds-of-types-in-scala-part-1/
- `abstract types` nearly like in OCaml
  ```scala
  trait AbstractModule {
    type T
    val value: T
  }

  trait ConcreteModule extends AbstractModule {
    type T = Int
    val value = 1
    // val value = "1" // error: type mismatch
  }

  (new ConcreteModule {}).value // 1
  ```
- `compound type` is mixed type `A with B with C ... { refinement }`
- `case classes` nearly like records in OCaml (can be matched and serialized)

## Collections
- `generic arrays` require run-time class tag through implicit context bound
  ```scala
  import scala.reflect.ClassTag
  def evenElems[T: ClassTag](xs: Vector[T]): Array[T]
  ```
- mutable and immutable list may be replaced just by changing `val` to `var`; operators and implicits makes other things
- `view` is lazy collection what executes transformations on its element only after materialization (e.x. `toVector`, `toList`)

## Other
- `for expression` is just new syntax, what generates code with map, flatMap, withFilter, foreach for classes with those methods
- `call-by-name` parameters are lazy and executes every time was accessed (similar to generator concept)
- treat `Single Abstract Method` types and Scala’s built-in function types uniformly from type checking to the back end
  ```scala
  scala> val r: Runnable = () => println("Run!")
  scala> r.run()
  // Run!
  ```
- functional pattern matching; `partial defined function` is a function, that invokes only after filtering via `isDefinedAt` function, which can be defined manually or with pattern matching:
  ```scala
  val second: List[Int] => Int = { case x :: y :: _ => y }
  // automatically translates to
  new PartialFunction[List[Int], Int] {
    def apply(xs: List[Int]) = xs match {
      case x :: y :: _ => y
    }
    def isDefinedAt(xs: List[Int]) = xs match {
      case x :: y :: _ => true
      case _           => false
    }
  }
  ```
  
  ## `cats`
  -  Cats generally prefers to use invariant type classes. This allows us to specify more specific instances for subtypes if we want.

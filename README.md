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
- `implitly[T]` means return implicit value of type T in the context
  ```scala
  implicit class Foo(val i: Int) {
    def addValue(v: Int): Int = i + v
  }

  implicit val foo: Foo = Foo(1)
  val fooImplicitly = implicitly[Foo] // Foo(1)
  fooImplicitly.addValue(1) // 2
    ```

## Modularity
- static module = `object` + `trait`
- dynamic module = `class` + `trait`
- https://github.com/yawaramin/scala-modules
- http://lambdafoo.com/scala-syd-2015-modules
- ⚠️ [DEPRECATED](https://dotty.epfl.ch/docs/reference/dropped-features/package-objects.html) `package objects` can contain arbitrary definitions, not just variable and method definitions. For instance, they are frequently used to hold package-wide type aliases and implicit conversions. Package objects can even inherit Scala classes and traits

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
- ⚠️ [DEPRECATED](https://dotty.epfl.ch/docs/reference/dropped-features/existential-types.html) `existential types`; use `type constraints` (`<:` or `:>`) with wildcards (`_`) instead
- `type refinement` is a type formed by supplying a base type with a number of members inside curly braces. The members in the curly braces refine the types that are present in the base type. For example, the type of “animal that eats grass” is `Animal { type SuitableFood = Grass }`
- `{ def close(): Unit }` is a `structural type`, because the base type is `AnyRef`, and `AnyRef` does not have a member named `close`; commonly used in conjunction with `type refinement`
- `Type alias` can be expressed as `infix` operator
  ```scala
    type <=[B, A] = A => B // invert type mapping
    type StringToIntFunction = Int <= String // same as String => Int
  ```

## Collections
- `generic arrays` require run-time class tag through implicit context bound
  ```scala
  import scala.reflect.ClassTag
  def evenElems[T: ClassTag](xs: Vector[T]): Array[T]
  ```
- mutable and immutable list may be replaced just by changing `val` to `var`; operators and implicits makes other things
  ```scala
  var imuSet = scala.collection.immutable.Set(1)
  val muSet = scala.collection.mutable.Set(1)

  imuSet += 2 // '+=' translates to 'imuSet = imuSet + 2'
  muSet += 2 // '+=' is mutable Set method
  ```
- `view` is lazy collection what executes transformations on its element only after materialization (e.x. `toVector`, `toList`)

## Parallelism/Concurrency
- avoid cyclic dependencies between `lazy` values, as they can cause deadlocks
- never invoke blocking operations inside `lazy` value initialization expressions or singleton object constructors
- never call `synchronized` on publicly available objects; always use a dedicated, private dummy object for synchronization

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
- `break` and `continue` is emulates with `breakable` + `break` methods
  ```scala
  def break(): Nothing = { throw breakException }
  def breakable(op: => Unit): Unit = {
    try {
      op
    } catch {
      case ex: BreakControl =>
        if (ex ne breakException) throw ex
    }
  }
  ```
  
## `cats`
-  `cats` generally prefers to use invariant type classes. This allows us to specify more specific instances for subtypes if we want.
- [`type Id[A] = A` type gives the ability to abstract over monadic and non-monadic code is extremely powerful.](https://books.underscore.io/scala-with-cats/scala-with-cats.html#sec:monads:identity) For example, we can run code asynchronously in production using Future and synchronously in test using `Id`.
- `cats` [`Eval` monad](https://books.underscore.io/scala-with-cats/scala-with-cats.html#eval-as-a-monad) makes computations stack safety (prevents stack overflow issue). For example a stack safety stream folding `Foldable[Stream]` (not safety for standard Scala stream folding via `Stream.foldLeft/foldRight`).

## Routine basics
- random numbers
  - https://stackoverflow.com/questions/39402567/get-random-number-between-two-numbers-in-scala
  - https://chrisalbon.com/scala/basics/random_integer_between_two_values/
- read/write from/to stdio, file
  - https://alvinalexander.com/scala/how-to-open-read-text-files-in-scala-cookbook-examples
  - https://alvinalexander.com/scala/how-to-write-text-files-in-scala-printwriter-filewriter
  - https://stackoverflow.com/questions/5055349/how-to-take-input-from-a-user-in-scala
- regexpr, interpolation
  - https://alvinalexander.com/scala/how-find-regex-patterns-matches-in-strings-scala
  - https://alvinalexander.com/scala/string-interpolation-scala-2.10-embed-variables-in-strings
  - https://alvinalexander.com/scala/how-find-replace-regex-patterns-in-scala-strings
- collections
  - [traverse with index](https://stackoverflow.com/questions/6821194/get-index-of-current-element-in-a-foreach-method-of-traversable)

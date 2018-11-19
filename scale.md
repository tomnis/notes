# immutable api with mutable internals
- yea mutable is immoral
- parser type class, mutable + immutable impls
- inspired by fastparse
- sealed trait with protected mutable internals
- parser examples
- write your combinators such that no allocation takes place
- 7-10x speedup using mutable internals
- "functional programming in the large, imperative in the small"
- design clear api, with a well-defined typeclass

# transpiling graphql
- resolvers: specify fetching data
- autogenerate resolvers
- no business logic in the api layer
- no code examples of how this works
- seems like walk the ast of graphql query, emit code for internal lang

# scala to bytecode
- how are case classes, lazy vals, higher order functions, etc implemented in jvm?
- jvm is agnostic to source that created the bytecode
- scalac -Xshow-phases
- for comprehensions expanded to flatmaps
- synthetic getters added
```
object Config {
  val a: String = "hi"
}
```
- config$.class (scala object) (contains public reference to the singleton)
- trait:
- compiles to an interface (with default impls) and abstract class
- another example:
```
def m(): Int = 3

val m1 = m
var m2 = m
def m3 = m
lazy val m4 = m
```
- references are stored in the constant pool
- what happens for the lazy val?
- another bitfield used to store the initialized state of m4
- lazycompute is invoked
- threadsafe: monitorenter and monitorexit instructions
- perf overhead of init is only on the first call
- 32 lazy values can be tracked in a single int with a bitmask
- Function2 -- scalac will generate permutations for primitives
- partially applied functions:
- scalac generates a new class for each
- first class/anon functions
- generate new class extending `Function1` with `apply` method


# haskell
- instance deriving show
- auto typeclass
- idea, given a basic structure we can have the compiler generate a lot of things for free
- (de)serialization, generating instance with random fields, swagger doc, mocked api
- test suite tests roundtripping of serialization process

# rsc (parallel scala compiler)
- parse, scope
- once the type signatures (outlines) are known, rest can happen in parallel
- typechecking is 25x faster than scalac
- emit scalasignature (intermediate format), and invoke multiple instances of scalac
- test benchmark on github/twitter/util (75k loc)

# unison lang
- scalaing, deployment, provisioning are part of the programming model

# sourced.tech
- ml on source code
- assisted code review
- removing a token from file, predict the missing token
- for code review, predict the blocks that are "most interesting" or "most surprising"


# EitherT
- Either[L, R] (right biased (left is error case))
- val right: Either[String, Int] = Right(1)
- monad (can used flatmap (for comprehensions))
- EitherT[Future, L, R]
- parameterized by a wrapper type
- useful for replacing Future[Either[L, R]]
- leftMap for transforming errors
- monad transformer

# teaching scala at twitter
- teaching is part of engineer responsibilities
- getting peers to teach leads to lots of idiom sharing

# shapeless
- circe, doobie, spark (frameless) use it
- ex: labelling emails, look at _pairwise_ agreement of data annotators
- can be a for-comprehension, or possibly std lib combinations function
- compiler has unique types for literal values
- final val t = 3
- however, if we use combinations, we get a List[List[T]], when ideally for more safety we would like a List[Tuple2[T, T]]
- instead of directly using the result of std lib combinations,
- take advantage of the literal value and map to HList, then tupled
- we retain enough information to reconstruct in a typesafe way
- replicateH, repeating the effect of an applicative functor
- kittens!
- shapeless and cats can be lots of typenoise, but the actual implementations are trivial
- if you find yourself wishing you had more of the info that was available at compile time, shapeless can likely help
- multihour compile times LOL (in one use case prior to a scalac patch)


# duality
...
- closely related to recursion schemes
- elgot algebras
- comonadic folds
- recall reversing all the arrors
- abstracted notion of a function (kleisli arrow)
- by specializing on kleisli, we get cata for free
- moving everything into type parameters, we can delete a lot of code
- define Op (opposite) as a type alias that reverses all the arrows
- kind polymorphism -- specifying when different type parameters conform to the same 'shape'
- `F[_ <: AnyKind]`
- we can reduce our library to a single function


# radix trees
- see okasaki paper
- see adaptive radix trees paper
- tradeoff between tree span and height
- lesson: algebraic data types and pattern matching let the compiler assist you in getting tricky logic correct
- tries are good for sorted keys, prefix operation, and merging (like hierarchical config, eg typesafe config library)

# quantum state
- simulator for quantum computer implemented in scala
- represented as a monoid over complex numbers
- quantum bias is like partial reflection in a window
- example: 2 quantum coins. can't think of the coins independently, the result space is a tetrahedron
- github/logicalguess/quantum-scale
- github/jiszka/quantum-probability-monad


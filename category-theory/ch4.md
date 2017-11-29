- not a pure function, if the memoized version would fail to produce a log
- writer. in general, embellishing the return type of a bunch of functions
- embellishment corresponds to endofunctor

----

kleisli category for partial functions (is it flatMap?) f . g
- execute g to get an option
- match on g result, pass it to f, or short circuit

```scala
scala> implicit class KleisliCompose[B, C](val f: B => Option[C]) {
     |   def o[A](g: A => Option[B]) = (a: A) => g(a) match {
     |     case Some(b) => f(b)
     |     case None => None
     |   }
     | }
     
     
scala> val maybeSqrt = (x: Double) => if (x >= 0) Option(math.sqrt(x)) else None

scala> val maybeReciprocal = (x: Double) => if (x != 0) Option(1.0 / x) else None

scala> val h = maybeSqrt o maybeReciprocal
```


     

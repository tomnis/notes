- gregor samsa broke the type system
- types are sets of values (what makes this tricky?)
- bottom extends every type with one or more special value
	- runtime errors
	- what does this imply for the empty set type, haskell Void?

- operational vs denotational semantics
	- running program through interpreter, vs mathematical interpretation for every programming construct

- Void is not inhabited by any values (except for bottom?), corresponds to the empty set

- absurd can never be called, but has no restrictions on its return type
	- what does he really mean by this? i can define a function accepting an Int that returns whatever type i want


```scala
  class Memoizer[A, B] {
    import scala.collection.mutable
    val cache: mutable.Map[A, B] = mutable.Map.empty
    
    def memoize(f: A => B): A => B = {
      (a: A) => this.cache.get(a) match {
        case Some(b) => 
          b
        case None =>
          val b = f(a)
          this.cache.put(a, b)
          b
      }
    }
  }
```
	
	


- objects, arrows (morphisms), must compose
- composition should be associative.
- need identity function
- composition is the essence of programming
- surface area of programs should increase slower than their volume, so to speak

```scala
  def compose[A, B, C](f: A => B, g: B => C): A => C = {
    (a: A) => g(f(a))
  }
```

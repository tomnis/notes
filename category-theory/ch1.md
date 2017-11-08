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

```scala
  implicit class DecoratedFunction[B, C](val f: B => C) {

    def o[A](g: A => B): A => C = (a: A) => f(g(a))
  }


  val g = (a: Int) => a.toString
  val f = (s: String) => s.isEmpty

  val h: Int => Boolean = f o g

  val b: Boolean = h(5)
  ```

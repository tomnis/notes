# functors

- a mapping between categories, maps objects and morphisms
- preserves identity and composition
- endofunctor maps a category to itself
- maybe is a type constructor
- fmap lifts a function
- prove laws using equational reasoning
- haskell uses typeclass
  ```
  class Functor f where
    fmap :: (a -> b) -> f a -> f b
  ```

- list is a functor (any type parameterized by another type is a functor candidate)
  ```
  instance Functor List where
    fmap _ Nil = Nil
    fmap f (Cons x t) = Cons (f x) (fmap f t)
  ```

- `(->) a` is a type constructor (partially applied function)


- functors have all the properties of morphisms in some category:
  - there is an identity functor (trivial)
  - functor composition is associative

 - the category of categories

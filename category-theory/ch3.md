- free category: add edges to represent all the morphisms
- preorder: <=
  - satisfies identity, composition, associativity
  - (there is at most one morphism from any a to b) "thin category"
- partial order: `a <= b && b <= A -> a == b`
- total order: any 2 objects are in a relation with each other
- `hom-set`: `C(a, b)` = set of morphisms from a to b

- monoid: a set with an associative binary operation and a neutral element
```
class Monoid m where
  mempty :: m
  mappend :: m -> m -> m    // (m -> (m -> m))
```
- why can't we express the monoidal properties of `mempty` and `mappend`?
- we can express equality of functions, different from equality of return values
- point free (the value of f at x)
- c++, monoid concept is a predicate. test whether there are `mempty` and `mappend`
  for a given type

- monoid is a category (single object category)
- for any `n`, there is a function for adding `n`, the `adder`
- set of all adders is `homset(m, m)`
- ie, a single object with a bunch of morphisms, eg adders for addition
- we can always extract a set from a single object category (the set of morphisms)
- morphisms need not be functions!!

----
- 2) partial orders
- 3) (Bool, AND) forms a monoid. AND is associative. true is the neutral element.
- 4) (Bool, AND) as a category. morphisms = (AND True), (And False)
     rules of composition: 
       (AND True) . (AND True) = AND True
       (AND True) . (AND False) = AND False
       (AND False) . (AND True) = AND False
       (AND False) . (AND False) = AND False

- addition modulo 3: object = { 0, 1, 2 }. morphisms = { (+ 1), (+ 2), (+ 3) }

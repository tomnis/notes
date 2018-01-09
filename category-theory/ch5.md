# products and coproducts

- define objects in terms of their relationships
- initial object -- has one and only one morphism going to any other object
- uniqueness up to isomorphism
- terminal object -- one and only one morphism coming to it from any object
- we can always reverse the arrows to get a new category
- `product` of 2 objects `a` and `b` is the `c` equipped with 2 projections
  such that for any other object `c'` equipped with 2 projections, there is a unique
  morphism `m` from `c'` to `c` that factorizes these projections
- coproduct: reverse the arrows to get injections
- `coproduct` of 2 objects `a` and `b` is the `c` equipped with 2 injections such that
  for any other object `c'` equipped with 2 injections there is a unique morphism `m`
  from `c` to `c'` that factorizes the injections (disjoint union)

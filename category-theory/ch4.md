- not a pure function, if the memoized version would fail to produce a log
- writer. in general, embellishing the return type of a bunch of functions
- embellishment corresponds to endofunctor

----

kleisli category for partial functions (is it flatMap?) f . g
- execute g to get an option
- match on g result, pass it to f, or short circuit

this document summarizes my reading of essential slick.

most of the code examples here are based on the code that ships with the book
https://github.com/underscoreio/essential-slick-code

slick is not an orm. instead, slick attempts to provide abstractions that feel more
database-like.
the basic approach of slick is to treat database tables as if they were scala collections.
this provides an immediately familiar api for interacting with the tables (scala standard combinators)

monadic properties of `Query` and `DBIO` are crucial. we can reuse basic building blocks to create
more complex database interactions. a common pattern i've found myself using. for example,
we have a lot of findOrCreate logic thats basically the same for several different tables.
instead of duplicating the same logic in several different methods, we can make our own
combinator that accepts a find query and a create query, composes them to create a new
database action, and runs that action within a transactional boundary. this was always tricky
to get right, but query composition makes it much easier to reason about complex database interactions

- key types:
  - `Query` (compose using standard combinators map, flatMap, filter)
    used to build sql for a single query
  - `DBIOAction`/`DBIO` (things you run against db. these compose also. you can convert a query to an
    action by calling the result method)
    used to build sequences of sql queries, delineate transactions
  - `Future` (result of an action, also have combinators)
    transform the asynchronous result of running an action

- lifted embedding:
  - define data types to store row data (case classes)
  - define `Table` objects for mappings between our data types and the db
  - define `TableQuery` and combinators for composing useful queries

- recreating the schema (`Tables.schema.create`) from case classes actually recreates the uniqueness constraints.
  we always had a problem with this when recreating the schema from generated jpa classes
  ```
  val action: DBIO[Unit] = Tables.schema.create
  val future: Future[Unit] = db.run(action)
  val result = Await.result(future, 2 seconds)
  ```

- `Query` is a monad, composable with for comprehensions (aliases for chains of method calls)
- `DBIO` (action) is also a monad (`andThen`, `>>` combinators)
  ```
val actions: DBIO[Seq[Message]] = (
  messages.schema.create andThen
  (messages ++= freshTestData) andThen
  halSays.result
)

val actions = (
  messages.schema.create >>
  (messages ++= freshTestData) >>
  halSays.result
)
  ```
- selecting data
  - `lazy val messages = TableQuery[MessageTable] // select * from message`
  - `messages filter { _.sender === "HAL" } // select * from message where sender = "HAL"`
  - note the use of `===`. this is because slick lifts string, boolean, etc into a `Rep[String]`
  - `Query[M, U, C]`
  - `M` -> mixed type, parameter type of calling combinators like map or filter
  - `U` -> unpacked type (the type of the results)
  - `C` -> collection type that results are accumulated into (Seq or List)
  - `Query` companion object can be useful for confirming connectivity: `Query(1) // select 1`
  - use map combinator on a `Query` to select specific columns `messages map { _.content }`
  - `SELECT content FROM message` == `messages.map(_.content).result.statements`
  - recommended to call `map` only as the final step in a transformation, because of mixed type `M`
  - can also select multiple columns into tuples: `messages.map(t => (t.id, t.content))`
  - basically, use `map` to control the `SELECT` portion of the query
  - before running a `Query`, it must be converted to an action (sequence of queries)
  - typically the conversion is done by calling `query.result`
  - `DBIOAction[R, S, E]`
  - `R` -> type returned by the database (result)
  - `S` -> whether the results are streamed
  - `E` -> inferred effect type (eg `Read with Write with Transactional`)
  - these types can get pretty hairy, better to omit the type annotation and alt + enter
  - or, use `DBIO[T]` (alias for `DBIOAction[T, NoStream, Effect.All]`)

  - to execute an action, pass it to either `run()` or `stream()`
  - `run()` returns a `Future`
  - `stream()` returns a `DatabasePublisher` rather than a `Future`, i havent played with this
  - `DatabasePublisher` exposes `subscribe`, `map` (returns a new publisher), and `foreach` (side effects)

  - checking equality:
      - `===`, `=!=`, `<`, `>`, `<=`, `>=` (defined on `Rep`)
      - `++` method for string concat
      - numeric methods ceil, floor, round, arithmetic operators
      - `like` method for sql pattern matching: `messages filter { _.content like "%abcd%" }`

  - nullable columns of `T` are represented as `Option[T]`
  - `drop` and `take` can be used like sql limit and offset (pagination)

- inserting, deleting, and updating data
  - `+=` method creates a `DBIO` we can run (returns number of rows inserted)
  - `val action = messages += Message(...)`
  - autoinc keys
    - this option can be set on the columns in a Table class `O.PrimaryKey, O.AutoInc`
    - the corresponding case class field should have a default. the AutoInc option states that slick should ignore this field when inserting.
    - to override this, use `forceInsert` instead of `+=`
    - `val action = messages forceInsert Message(..., id = 666L)`
    - we often want to get back the id allocated by the db, use `returning` for this
    - `val insert: DBIO[Long] = messages returning messages.map(_.id) += Message(...)`
    - to get back the complete row with its newly allocated id, use `returning ... into`
    - `into` specifies a function used to combine the record and the new primary key

    ```
messages returning messages.map(_.id) into { (message, id) =>
    message.copy(id = id)
} += Message(...)
    ```
    
  - inserting only specific columns:
      - use `map` before calling `+=`
      - `messages map { _.sender } += "Bob"`
      
  - inserting multiple rows:
    - use `++=` for batch inserts
    - `messages ++= Seq(Message(...), Message(...), Message(...))`

  - deleting:
      - specify which rows to delete using `filter`, then call delete
      - `messages.filter(_.sender === "HAL").delete`

  - updating:
    - use `filter` to specify which rows, then `map` to specify which columns, then call `update` with new values
    - `messages filter { _.sender === "Bob" } map { _.sender } update "Robert"`
    - updating multiple fields:
    - use `map` to select tuple of columns, call `update` with a tuple of values

- combining actions
  - just like queries, actions can be composed using combinators:
  - `val action: DBIO[T] = action1 andThen action2 andThen action3`
  - `val action: DBIO[T] = DBIO.seq(action1, action2, ...)`
  - !! IMPORTANT: composing actions doesnt automatically give you a transaction
  - for a transaction, you need to `db.run(actions.transactionally)` (can also specify isolation level)

  - actions representing simple values (useful for flatmapping actions):
    - `DBIO.successful(200)`
    - `DBIO.failed(new RuntimeException(...))`
  
  - `DBIO.flatMap`
    - accepts a function f that accepts a value from an action, and returns another action
    - arbitrary action control flow
    - signature: `def flatMap[S](f: R => DBIO[S])(implicit e: ExecutionContext): DBIO[S]`
  - `DBIO.sequence`
    - accepts a sequence of `DBIO` and returns a `DBIO` of a sequence

  - `DBIO.fold`
    - used to combine actions such that the results are combined by the supplied function

  - `DBIO.zip`
    - combines actions such that the results of each action are returned in a tuple

  - `cleanUp`, `andFinally`
    - sort of like catch/finally
    - `cleanUp` accepts an `Option[Throwable]` (error handling)
    - `andFinally` can be used to close any connections/resources

  - transactions
    - not sure what the default level is
    - `db.run(action.transactionally.withIsolationLevel(Serializable))`
    - if any of the individual actions fail, will be rolled back
    - need retry logic


- modelling (advanced)
  - common supertype trait for database drivers: `slick.driver.JdbcProfile`
  - to write code that works for different databases (ex: mysql in prod, h2 in tests), import trait and inject a concrete impl
  
  - for representing individual rows, slick can use tuples, case classes, or `HList` (heterogenous list)
  - prefer case classes.
     - compared to tuples, they have named fields for better readability
     - `User(1L, "tomas").name` vs `(1L, "tomas")._1`
     - case classes have types, distinguish from other case classes with same field types
  - `HList` (heterogenous list, similar to tuple but not limited to 22 fields

  - projections
      - a projection specifies a mapping between db columns and scala values
      - when declaring a table, we need a method `*` (default projection)
      - however, in the base Table class, `*` has the type `ProvenShape` (not a tuple of columns)
      - `ProvenShape` is implicitly built from what the user provides (column, tuple)
      - `<>` operator allows converting between different types of `ProvenShape`
      - `<>` accepts 2 conversion functions, 1 for each direction
      - thus, as long as we can specify a way to convert a tuple to an `A`, we can store instances of `A` in db

  - working with nullable collumns
    - represented as `Option[T]` in case classes
    - but, be careful about selecting:
      - `users.filter(_.email === None)` // bad, sql expression "email" = null always evals to null
      - `users.filter(_.email.isEmpty)` // good, will use sql IS NOT NULL syntax

  - more on autoinc:
    - set the autoinc option on the column definition in the Table class
    - `def id = column[Long]("id", O.PrimaryKey, O.AutoInc)`
    - autoinc primary keys are often represented as with a default in the case classes
    - `case class User(name: String, id: Long = 0L)`
    - another possibility is to represent id as an `Option`
    - personally i dont think this is a good idea, we're not trying to encode that id can be null in the db

  - compound primary keys, indices, foreign keys
    - use `primaryKey` to encode compound keys
    - `def pk = primaryKey("room_user_pk", (roomId, userId))`
    - use `index` to define indices
    - `def nameIndex = index("name_idx", name, unique=true)`
    - use `foreignKey` to define forgein keys
    - `def sender = foreignKey("sender_fk", senderId, users)(_.id)`
    - this is a crucial difference from jpa, our generated entities didn't include indices
    - unlike with jpa, recreating schema from generated code actually works

  - other column tricks (advanced)
    - see `ColumnOption`, in addition to `PrimaryKey`, `AutoInc`, also has `Length`, `DBType`, `Default`
    - custom column mappings, for example can map sql timestamps to joda types
    - use value classes to model primary keys
    - this can prevent bugs like using the wrong field that has the correct expected type (like mistakenly providing a user id where
    a message id is expected
    - compile-time wrapper around the key value (no runtime overhead)
    - `case class MessagePK(value: Long) extends AnyVal with MappedTo[Long]`
    - sum types (disjunction)
       - example: flag messages as important/offensive/spam (mutually exclusive)
       - suppose we are encoding flags as a single char in the db schema
       - create a sealed trait `Flag` /case objects to model the flags
       - add an `Option[Flag]` field to case class
       - create a new custom `ColumnType` that includes bidirectional mapping Flag <=> Char

- joins (monadic and applicative)
  - applicative uses join method, while monadic uses flatMap (for comprehensions)
  - `messages join users on (_.senderId === _.id) // applicative join`
  ```
for { // monadic join
  msg <- messages
  usr <- users if usr.id === msg.senderId
} yield (usr.name, msg.content)
```
  - `join` => inner join
  - `joinLeft` => left outer join
  - `joinRight` => right outer join
  - `joinFull` => full outer join
  - omit `on <condition>` for an unconstrained cross join
  - generated sql queries can be overly complicated and slow (on mysql, postgres is better)
  - if your monadic joins are slow, try applicative joins, or drop into raw sql

- aggregate functions
  - `length`, `countDistinct`, `min`, `max`, `sum`, `avg`
  - `messages.length.result // select count(*) from messages`
  - `groupBy` (must be followed by a map)
  ```
val msgPerUser = messages.groupBy(_.senderId).map { case (senderId, msgs) => senderId -> msgs.length }. result
  ```      

- plain sql
  - sql string interpolators can be used to build `Query` from raw strings
  - no need to worry about sql injection
  - `sql"select count(*) from messages"`
  - but, there is some loss of type safety
  - `tsql` interpolator connects to db at compile time to typecheck the queries
  - no guarantee that the schema in the db isn't modified before runtime, but this is a good idea
  - i'm not sure about compile time performance hit of this extra db connection
  - `sqlu"select count(*) from $table"` // wont work, the expanded value of table is quoted
  - instead, use `#$table` (splicing)
  - !!! IMPORTANT splicing doesnt protect from sql injection, never use splicing with user-supplied data

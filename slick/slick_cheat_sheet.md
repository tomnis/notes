quick reference guide for using slick.
  
- basic slick concepts (all of these are monads and can be manipulated/composed with the above combinators):
  - `Query`
    - monad representing sql for a single query
    - convert to an action by calling result `val action = messages.length.result`
  - `DBIO (action)`
    - monad for representing a sequence of sql queries. these are what is actually run against the db
    - `val action: DBIO[T] = action1 andThen action2 andThen action3`
    - `val future: Future[T] = db.run(action)`
    - !!! composed actions are not automatically transactional, for that use `db.run(action.transactionally)`
  - `Future`
    - result of an async db interaction
    - `Await.result(db.run(action), 2 seconds)`
  
- selecting data:
  - use `map` to control the "select" part of your query (column projection)
  - `messages.map(t => (t.id, t.content)) // select id, content from message`
  - use `filter` to control the "where" clause of your query (note `===`)
  - `messages filter { _.sender === "HAL" } // select * from message where sender = "HAL"`
    
- writing data:
  - use `+=` for a single row, `++=` for batch insert
  - `val action = messages += Message(...)`
  - `val action = messages ++= Seq(Message(...), Message(...), Message(...))`
  - to get back the newly allocated autoinc id, use `returning`
  - `val insert: DBIO[Long] = messages returning messages.map(_.id) += Message(...)`
  - to get back a new case class with the newly allocated id, use `returning ... into`
  - `messages returning messages.map(_.id) into { (message, id) => message.copy(id = id) } += Message(...)`
  - to update, use `filter` to specify rows, `map` to specify columns, and call `update` with new values
  - `messages filter { _.sender === "Bob" } map { _.sender } update "Robert"`
  
- joins (applicative, monadic)
  - applicative uses explicit functions `join`, `joinLeft`, etc
  - `messages join users on (_.senderId === _.id)`
  - monadic uses for comprehensions (alias for nested `flatMap`)
  ```
for { // monadic join
    msg <- messages
    usr <- users if usr.id === msg.senderId
} yield (usr.name, msg.content)
  ````
  
  
  

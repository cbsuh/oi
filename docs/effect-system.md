# Effect System

In `oi`, "Effect Transparency" is a core principle. Functions cannot unexpectedly perform I/O, mutate global state, or halt the program without documenting those behaviors in their type signature. 

## The `with` Keyword
Every side-effecting capability is represented as an *Effect*. When a function requires an effect, it declares it using the `with` keyword.

### Available Effects
While custom effects might be considered later, standard ones include:
- `io`: File, Network, and standard I/O streams.
- `db`: Database access or transactions.
- `async`: For asynchronous coroutine suspension.
- `log`: Telemetry and logging.
- `crypto`: Cryptographic operations requiring entropy.

### Custom Effects (Planned)
Custom effects will be built on top of the `trait` system (see ADR-0004). A custom effect is essentially a trait that defines a set of capabilities a function requires from its environment.

```oi
-- Possible future syntax (not yet finalized):
pub trait Storage {
  fn read(key: String) -> Result[Bytes, StorageError]
  fn write(key: String, value: Bytes) -> Result[(), StorageError]
}

fn cache_user(user: User) -> Result[(), StorageError]
  with Storage
{
  Storage.write(user.id.to_string(), user.serialize())?
  Ok(())
}
```

> **Note**: Whether custom effects reuse the `trait` keyword or introduce a dedicated `effect` keyword will be decided during implementation (Phase 2). The distinction matters if advanced features like effect handlers are introduced later.

## Pure vs Effectful Functions
- **Pure Functions**: A function without a `with` clause is strictly pure. Given the same inputs, it guarantees the same outputs and performs no side-effects. This means AIs can safely cache, replace, or optimize them.
- **Effectful Functions**: They document their interaction with the real world cleanly without hidden magic.

## Replacing Monad Transformers
In languages like Haskell, handling multiple effects (errors, state, IO, configs) leads to complex monad transformer stacks. This heavily degrades readability, requires intricate "lift" handling, and confuses AI models.

**Haskell Example (The Problem):**
```haskell
-- Haskell requires stacking effects, making order crucial and execution unclear.
type AppM a = ReaderT Config (StateT AppState (ExceptT AppError (WriterT [Log] IO))) a
getUser :: UserId -> AppM User
getUser uid = do
  config <- ask
  modify (\s -> s { queryCount = queryCount s + 1 })
  tell [Log "Fetching user"]
  result <- liftIO (dbQuery config uid)
  case result of
    Nothing -> throwError (UserNotFound uid)
    Just u  -> return u
```

**oi Example (The Solution):**
In `oi`, you simply list the required capabilities. There is no stack order, no `lift` operations, and errors are cleanly propagated inline.

```oi
fn get_user(uid: UserId) -> Result[User, AppError]
  with io, db, log, config
{
  log.info("Fetching user")
  let user = db.query(config.db_url, uid)?
  Ok(user)
}
```

## Why it matters for AI-human pairing
When an AI generates a function `fn calculate_tax(amount: Float) -> Float`, a human reviewer can instantly trust that this function does not secretly log data or access a database because it lacks `with io` or `with db`. Furthermore, the AI knows exactly the scope of side effects based on signature matching.

# Effect System

In `oi`, "Effect Transparency" is a core principle. Functions cannot unexpectedly perform I/O, mutate global state, or halt the program without documenting those behaviors in their type signature. 

## The `with` Keyword
Every side-effecting capability is represented as an *Effect*. When a function requires an effect, it declares it using the `with` keyword.

### Standard Effects

oi provides the following standard effects (ADR-0018):

**Independent effects:**
- `db`: Database access or transactions.
- `log`: Telemetry and logging.
- `crypto`: Cryptographic operations requiring entropy.
- `spawn`: For creating and communicating with actors (ADR-0013).
- `concurrent`: For structured parallel execution of independent tasks (ADR-0013).

**Fine-grained I/O effects:**
- `network`: HTTP, TCP, UDP, DNS operations.
- `fs`: File system read/write.
- `stdout`: Standard output.
- `stderr`: Standard error.

**Convenience group:**
- `io`: Equivalent to `network + fs + stdout + stderr`. Use for scripts, `main()`, and prototyping. In production code, prefer fine-grained effects.

```oi
-- Fine-grained (production code)
fn fetch(url: String) -> Result[String, HttpError]
  with network
{ ... }

fn read_config(path: String) -> Result[Config, IoError]
  with fs
{ ... }

-- Convenience group (scripts, main)
fn main()
  with io                    -- io ⊃ network, fs, stdout, stderr
{
  let data = fetch("https://api.example.com")?   -- ✅ io includes network
  io.println(data)
}
```

### Effect Propagation

Effects propagate **manually with compiler verification** (ADR-0018). Every function must declare all effects it uses, including transitive effects from callees. The compiler emits an error if an effect is missing.

```oi
fn save_user(user: User) -> Result[Unit, DbError]
  with db
{
  db.insert(user)
}

fn register(name: String) -> Result[Unit, DbError]
  with db              -- Required: save_user needs `db`
{
  let user = User.new(name)
  save_user(user)?
}

-- If `with db` is omitted from register():
-- ERROR: register() calls save_user() which requires `db` effect
```

> **Rule**: A function's `with` clause must be the **complete** set of effects it requires. The compiler verifies this against all callees. No automatic inference — visibility is the design goal.

### Effect Hierarchy Rules

- `with io` satisfies callees that require `network`, `fs`, `stdout`, or `stderr`.
- `with network` does **not** satisfy callees that require `fs` — they are independent.
- Non-I/O effects (`db`, `log`, `crypto`, `spawn`, `concurrent`) are always independent.

```oi
fn process()
  with db, network     -- ✅ db is independent, network is fine-grained
{ ... }

fn main()
  with io, db          -- ✅ io covers network/fs/stdout/stderr; db is separate
{
  process()?           -- ✅ io includes network, db is declared
}
```

### Custom Effects (Deferred to Phase 2+)

Custom effect syntax is deferred until Phase 2 implementation provides experience to evaluate the design trade-offs. The choice between reusing `trait` or introducing a dedicated `effect` keyword remains open (ADR-0018).

```oi
-- Possible future syntax (not yet finalized):
-- Option A: trait-based
pub trait Storage {
  fn read(key: String) -> Result[Bytes, StorageError]
  fn write(key: String, value: Bytes) -> Result[Unit, StorageError]
}

-- Option B: effect keyword
pub effect Storage {
  fn read(key: String) -> Result[Bytes, StorageError]
  fn write(key: String, value: Bytes) -> Result[Unit, StorageError]
}
```

> **Note**: The standard effects listed above are sufficient for Phase 2. Custom effects will be designed when implementation experience reveals the right abstraction.

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
  with db, log, network
{
  log.info("Fetching user")
  let user = db.query(uid)?
  Ok(user)
}
```

## Why it matters for AI-human pairing
When an AI generates a function `fn calculate_tax(amount: Float) -> Float`, a human reviewer can instantly trust that this function does not secretly log data or access a database because it lacks `with io` or `with db`. Furthermore, the AI knows exactly the scope of side effects based on signature matching.

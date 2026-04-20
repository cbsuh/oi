# Comparison with Other Languages

## Overview

`oi` is not built from scratch — it stands on the shoulders of proven languages. Its closest ancestor is **Elixir/Erlang**: the actor model, per-process GC, immutability-first design, and pipeline-oriented data flow all come directly from the BEAM ecosystem. Where `oi` diverges is in addressing Elixir's practical pain points — dynamic typing, invisible side effects, inconsistent error handling — by reinforcing the same philosophy with static types, an effect system, and Design-by-Contract.

From other languages, `oi` adopts Rust's type-safe error handling, Go's one-canonical-form ethos, and Eiffel's contract system.

## What we Inherited

| Source | What we took | oi Counterpart |
|--------|-------------|----------------|
| **Elixir / Erlang** | Actor-based concurrency, message passing | `actor` + `spawn` (ADR-0013) |
| **Elixir / Erlang** | Per-process GC, isolated heaps | Identical architecture (ADR-0012) |
| **Elixir / Erlang** | Immutability by default | `let` by default, `var` local-only |
| **Elixir / F#** | Data pipeline `\|>` operator | Identical adoption |
| **Rust** | `Result`-based error handling, `?` operator | Identical adoption |
| **Rust / ML** | ADTs, Exhaustive Pattern Matching | Identical adoption |
| **Go** | One Canonical Form for formatting | Built into the language spec |
| **Eiffel / Spark** | Design by Contract (`requires`/`ensures`) | Core syntax primitives |

## What we Fixed (vs. Elixir)

Elixir got the big architectural decisions right. But real-world Elixir development has recurring friction points that `oi` addresses:

| Elixir's Weakness | The Problem | oi's Fix |
|-------------------|-------------|----------|
| **Dynamic typing** | Types checked at runtime. Dialyzer is optional, slow, and its error messages are notoriously cryptic. AI cannot catch type mismatches at generation time. | Static type system with generics and trait-based polymorphism. |
| **Error handling duality** | Three coexisting patterns: `{:ok, val}`/`{:error, reason}` tuples, `!` bang functions that raise, and `with` macro chains. No single canonical way. | `Result[T, E]` + `?` operator. Exceptions do not exist. One way only. |
| **Invisible side effects** | Any function can freely perform IO, DB access, or network calls. The function signature reveals nothing about side effects. | `with` effect declarations. A function without `with` is guaranteed pure. |
| **No contracts** | No built-in mechanism to express preconditions or postconditions. Guards are limited to basic type checks and a few operators. | First-class `requires`/`ensures` in function signatures. |
| **Macro-driven DSLs** | Phoenix, Ecto, and Absinthe each introduce their own macro-based DSL. Code often looks nothing like standard Elixir, requiring per-library learning. | No macro system. Structured metadata (`---` blocks) and annotations (`@`) handle metaprogramming needs. |

## What we Discarded

| Discarded Feature | The Core Problem | oi's Replacement |
|-----------|----------|---------------|
| **Monad Transformers** | Causes order-dependent semantics, `lift` spaghetti, and confusing errors. | Simply list side-effects with `with`. |
| **Type-Level Metaprogramming** | Creates a secondary "invisible" language, leading to cryptic compilation errors. | Generics paired with runtime Contracts. |
| **Lazy Evaluation** | Can cause massive memory leaks (Space leaks) and makes execution order unpredictable. | Strict evaluation by default, isolated `lazy` scopes. |
| **Enforced Point-free Style** | Destroys readability; human and AI cannot deduce data intent. | `\|>` Pipelining and explicit bindings. |
| **Implicit Typeclass Resolution**| Behavior heavily depends on imports. It executes "hidden code" globally. | Explicit interfaces and injections. |
| **Dependent Types** | Math theorem proving is required. Undecidable type inference. | Simple `requires` / `ensures` runtime assertions. |

## Detailed Comparisons

### vs. Elixir
- **Relationship**: Elixir is `oi`'s closest ancestor. The actor model, per-process GC, immutability-first design, and pipeline operator all originate from the BEAM ecosystem.
- **What oi fixes**: Static types, unified error handling, effect tracking, and Design-by-Contract address Elixir's practical friction points.

**Error handling — three ways vs. one way:**
```elixir
# Elixir: three coexisting patterns for the same operation

# Pattern 1: tuple matching
case Repo.get(User, id) do
  nil -> {:error, :not_found}
  user -> {:ok, user}
end

# Pattern 2: bang function (raises on failure)
user = Repo.get!(User, id)

# Pattern 3: with macro
with {:ok, user} <- fetch_user(id),
     {:ok, order} <- create_order(user) do
  {:ok, order}
end
```

```oi
-- oi: one way only
fn find_user(id: UserId) -> Result[User, UserError]
  with db
{
  let user = db.query(id)?
  match user {
    Some(u) => Ok(u)
    None    => Err(UserError.NotFound { id })
  }
}
```

**Side effects — invisible vs. declared:**
```elixir
# Elixir: this signature reveals nothing about what the function touches
def process_payment(order_id) do
  order = Repo.get!(Order, order_id)     # DB
  Logger.info("Processing #{order_id}")  # Log
  PaymentGateway.charge(order.amount)     # Network
  Mailer.send_receipt(order.email)        # Network
end
```

```oi
-- oi: every side effect visible in the signature
fn process_payment(order_id: OrderId) -> Result[Receipt, PaymentError]
  with db, log, network
{
  let order = db.get(order_id)?
  log.info("Processing {order_id}")
  let receipt = network.charge(order.amount)?
  network.send_receipt(order.email)?
  Ok(receipt)
}
```

### vs. Rust
- **Similarity**: Both emphasize rigorous correctness (Result types, immutability by default).
- **Difference**: Rust is optimized for absolute memory control and performance at the cost of high cognitive overhead for developers. `oi` sacrifices micro-optimization features (like explicit lifetimes) for a simpler, "One Canonical Form" syntax. It replaces Rust's borrow checker mental overhead with higher-level semantic safety checks.

```rust
// Rust: lifetimes and borrow checker appear in the signature
fn find_user<'a>(db: &'a DbPool, id: UserId) -> Result<&'a User, AppError> {
    let user = db.query(id).map_err(|e| AppError::Db(e))?;
    user.ok_or(AppError::NotFound(id))
}
```

```oi
-- oi: no lifetimes, effects declared explicitly
fn find_user(id: UserId) -> Result[User, AppError]
  with db
{
  let user = db.query(id)?
  match user {
    Some(u) => Ok(u)
    None    => Err(AppError.NotFound { id })
  }
}
```

### vs. Go
- **Similarity**: Both value simplicity and explicit handling of errors over exceptions.
- **Difference**: Go requires extensive handling boilerplate (`if err != nil`). `oi` handles this more concisely via ADTs and `?`. Furthermore, Go allows side-effects everywhere implicitly; `oi` tracks them explicitly via the `with` keyword.

```go
// Go: if err != nil repeated for every fallible call
func processOrder(id int) (*Receipt, error) {
    order, err := fetchOrder(id)
    if err != nil { return nil, err }
    payment, err := chargePayment(order.Amount)
    if err != nil { return nil, err }
    receipt, err := generateReceipt(order, payment)
    if err != nil { return nil, err }
    return receipt, nil
}
```

```oi
-- oi: ? operator propagates errors in one line each
fn process_order(id: OrderId) -> Result[Receipt, OrderError] with db, payment {
  let order = fetch_order(id)?
  let payment = charge_payment(order.amount)?
  let receipt = generate_receipt(order, payment)?
  Ok(receipt)
}
```

### vs. Python
- **Similarity**: Fast iteration, readable pseudo-code feel.
- **Difference**: Python requires you to guess parameter types, return shapes, and whether a function drops a database table dynamically. `oi` enforces constraints strictly via its type system, ensuring that AI hallucinations don't produce invalid code.

```python
# Python: type hints optional, side effects invisible
def process_user(user_id):
    user = db.query(user_id)  # DB access? Can't tell
    send_email(user.email)     # Network call? Can't tell
    return {"status": "ok"}    # Return type? Dict? Can't tell
```

```oi
-- oi: types, errors, and side effects all explicit in signature
fn process_user(user_id: UserId) -> Result[Status, UserError] with db, network {
  let user = db.query(user_id)?
  network.send_email(user.email)?
  Ok(Status.Ok)
}
```

### vs. TypeScript
- **Similarity**: Structural typings providing gradual structuring.
- **Difference**: TypeScript sits on top of JavaScript, inheriting JS's implicit behaviors and quirks. `oi` is built from the ground up natively to prevent these historical artifacts, making it much safer for AI context windows where hallucinating a dynamic trait can break the build unseen.

```typescript
// TypeScript: runtime NaN on division by zero, no compiler warning
function divide(a: number, b: number): number {
  return a / b; // b === 0? Returns NaN silently
}
```

```oi
-- oi: contracts enforce preconditions, compiler verifies
fn divide(a: Float, b: Float) -> Float
  requires b != 0.0
  ensures result => result * b == a
{
  a / b
}
```

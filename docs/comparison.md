# Comparison with Other Languages

## Overview
How does `oi` compare to existing programming languages? As a language tailored for AI-human pairing, `oi` carefully borrows the best practices of modern languages while intentionally discarding certain paradigms.

## What we Adopted

| Source Concept | Feature | oi Counterpart |
|------|------|--------------|
| **Rust** | `Result`-based error handling, `?` operator | Identical adoption |
| **Rust / ML** | Algebraic Data Types (ADTs), Exhaustive Pattern Matching | Identical adoption |
| **Go** | A strict "One Canonical Form" for style formatting | Built directly into the language spec |
| **Elixir / F#** | Data pipeline `\|>` operator | Identical adoption |
| **Eiffel / Spark**| Design by Contract (`requires`/`ensures`) | Core syntax primitives |

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
fn find_user(id: UserId) -> Result[User, AppError] with db {
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

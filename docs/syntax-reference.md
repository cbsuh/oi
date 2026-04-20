# Syntax Reference

This document provides a high-level overview of the `oi` syntax. The syntax is designed to be highly structured and unambiguous.

## Modules and Imports
`oi` uses the `module` and `use` keywords. We avoid having aliases like `import`, `require`, or `include` to adhere to "One Canonical Form".

```oi
module UserService
  use core.{Result, Error}
  use db.Connection
```

## Visibility

Everything is **private by default**. Use the `pub` keyword to make items accessible from other modules.

```oi
module UserService
  use core.{Result, Error}

-- Private: only accessible within this module
fn validate_email(email: String) -> Bool { ... }

-- Public: accessible from other modules
pub fn create_user(name: String, email: String) -> Result[User, UserError] { ... }

-- Public type
pub type User {
  id: UserId
  name: String
  email: Email
}

-- Private type (internal to this module)
type InternalCache { ... }
```

**Rule**: If AI forgets `pub`, the item stays private (safe). This is intentional — exposing an API should be a conscious decision.

## Types
Types in `oi` are strictly defined and support Records, Algebraic Data Types (ADTs), Generics, and Structural Metadata annotations.

```oi
-- Record Type
type User {
  id: UserId
  name: String @max_length(100)
  email: Email @unique
  role: Role = Role.Member
  created_at: Timestamp @auto
}

-- Algebraic Data Type
type PaymentStatus =
  | Pending { amount: Money, due: Date }
  | Completed { amount: Money, paid_at: Timestamp }
  | Failed { reason: String, retry_count: Int }
  | Refunded { original: Completed, refund_at: Timestamp }

-- Standard ADTs
-- Result[T, E] = Ok(T) | Err(E)
-- Option[T] = Some(T) | None
```

## Traits and Interfaces

Polymorphism is expressed via `trait`. Types implement traits explicitly — there is no implicit structural matching.

### Defining a Trait
```oi
pub trait Comparable {
  fn compare(self, other: Self) -> Ordering
}

pub trait Printable {
  fn to_display(self) -> String
}
```

### Inline Implementation (for your own types)
When defining a type, implement traits inline using `with`:

```oi
pub type User {
  name: String
  email: Email
} with Printable {
  fn to_display(self) -> String {
    "{self.name} <{self.email}>"
  }
}
```

### Standalone Implementation (for external types)
For types you don't own, use a separate `impl` block:

```oi
impl Printable for Int {
  fn to_display(self) -> String {
    self.to_string()
  }
}
```

### Generic Constraints
Use traits to constrain generic type parameters:

```oi
fn sort[T: Comparable](list: List[T]) -> List[T] { ... }
fn print_all[T: Printable](items: List[T])
  with io
{ ... }
```

## Functions
Functions are defined using the `fn` keyword. Functions without `with` are guaranteed pure.

```oi
-- Pure function: no side effects
fn add(a: Int, b: Int) -> Int {
  a + b
}

-- Effectful function: side effects declared via `with`
fn save_user(user: User) -> Result[UserId, DbError]
  with db
{
  db.insert(user)
}
```

## Contracts (Pre/Post-conditions)
Functions can declare their operational constraints explicitly:

```oi
fn divide(a: Float, b: Float) -> Float
  requires b != 0.0
  ensures result => result * b == a
{
  a / b
}
```

## Effect Declarations
Side effects are declared using the `with` keyword. The `with` clause is always placed on its own line, indented by 2 spaces below the function signature (see ADR-0009).

```oi
fn print_message(msg: String)
  with io
{
  io.println(msg)
}
```

## String Interpolation
Strings support direct variable interpolation using `{var}` or `{expression}`. No special prefix like `f` or `$` is needed.

```oi
let name = "Alice"
let age = 30
let greeting = "Hello, {name}. Next year you will be {age + 1}."
```

Escaping literal braces is done with a backslash: `\{` and `\}`.

## Variables & Mutability

`oi` uses two keywords: `let` for immutable bindings and `var` for mutable bindings.

```oi
let x = 10       -- Immutable. x can never be reassigned.
var counter = 0   -- Mutable. counter can be reassigned within this function.
counter = counter + 1
```

### Local-Only Mutation Rule

`var` exists **only inside function bodies**. It cannot cross function boundaries:

```oi
-- ✅ Allowed: local var inside a function
fn sum_prices(orders: List[Order]) -> Money {
  var total = Money(0)
  for order in orders {
    total = total + order.price
  }
  total
}

-- ❌ Forbidden: var as a function parameter
fn process(var x: Int) -> Int { ... }  -- Compile error

-- ❌ Forbidden: module-level (global) var
module Counter
  var count = 0  -- Compile error

-- ❌ Forbidden: var in type fields
type User {
  var name: String  -- Compile error
}
```

**Why this design?** From the outside, every function in `oi` looks pure — its signature has no trace of internal `var` usage. AI and human reviewers can trust the signature. Inside the function, `var` provides practical convenience for loops and accumulators without compromising external safety.

## Collections & Indexing
Collections like `List` or `Map` use 0-based bracket notation `[]` for item access. 

```oi
let items = [10, 20, 30]
let first = items[0]
```
> **Note**: Even though `[]` is also used for Generics (`List[Int]`), there is no ambiguity because `oi` enforces that type names start with an uppercase letter, while identifiers and values do not.

## Conditional Expressions
`if`/`else` is an **expression** — it returns a value. Braces are always required. There is no ternary operator (`?:`) because `if`/`else` expressions make it redundant (see ADR-0014).

```oi
-- if/else returns a value
let category = if score >= 90 {
  "excellent"
} else if score >= 70 {
  "good"
} else {
  "needs improvement"
}

-- Single-line form (braces still required)
let sign = if x > 0 { "+" } else { "-" }

-- As the last expression in a function
fn abs(x: Int) -> Int {
  if x >= 0 { x } else { -x }
}

-- Without else: evaluates to Unit `()`
if should_log {
  log.info("event occurred")
}
```

### Control Flow Parentheses Rule
Control flow keywords (`if`, `match`, `for`, `while`) do **not** require parentheses around their condition or subject (see ADR-0014). Expression grouping parentheses are allowed but `oi-fmt` strips unnecessary outermost parentheses.

```oi
-- ✅ Canonical form
if age >= 18 { ... }

-- ⚠️ Valid but oi-fmt simplifies to the above
if (age >= 18) { ... }

-- ✅ Internal grouping parentheses are preserved
if (a > 1) || (a == -1) { ... }
```

## Loops

oi provides two loop constructs: `for` (collection/range iteration) and `while` (conditional repetition). Loops are **statements** — they always return Unit `()`. There is no `break` or `continue`; all early termination is handled by pipeline operations like `find`, `take_while`, and `filter` (see ADR-0015).

### `for` — Collection and Range Iteration

`for` iterates over every element in a collection or range. It always processes the entire sequence.

```oi
-- Iterate over a collection
for item in items {
  io.println(item.to_string())
}

-- Range syntax: .. (exclusive end), ..= (inclusive end)
for i in 0..10 {
  io.println(i.to_string())   -- 0, 1, 2, ..., 9
}

for i in 1..=5 {
  io.println(i.to_string())   -- 1, 2, 3, 4, 5
}

-- Destructuring
for (key, value) in entries {
  io.println("{key}: {value}")
}

-- Accumulation with var (ADR-0002)
fn sum_prices(orders: List[Order]) -> Money {
  var total = Money(0)
  for order in orders {
    total = total + order.price
  }
  total
}
```

### `while` — Conditional Repetition

`while` repeats its body as long as the condition is true. The loop exits **only** when the condition becomes false.

```oi
var retries = 0
var connected = false
while !connected && retries < max_retries {
  connected = try_connect().is_ok()
  if !connected { retries = retries + 1 }
}
```

### No `break` / `continue`

oi intentionally omits `break` and `continue` (ADR-0015). All loop control flow is structured:

```oi
-- ✅ Need early termination from a collection? Use pipeline.
let found = items |> find(|x| x.matches(query))
let valid = items |> filter(|x| !x.is_invalid())
let first_10 = items |> take_while(|x| x.score > 0)

-- ✅ Need conditional repetition? Use while with composed conditions.
var done = false
while !done {
  let result = process_next()
  done = result.is_complete()
}

-- ✅ Need infinite event processing? Use actors (ADR-0013).
actor Listener {
  on Message { data: String } {
    process(data)
  }
}
```

> **Rule**: `for` = iterate entire collection. `while` = repeat until condition is false. For early termination, use `|>` pipeline operations. For infinite loops, use `actor`.

## Pattern Matching
Pattern matching replaces complex `if-else` chains. Exhaustive matching is required. You can add extra conditional logic using `if` guards.

```oi
match value {
  1 => "One"
  2 => "Two"
  x if x > 10 => "Greater than ten"
  _ => "Other"
}
```

`match` is an **expression** — it evaluates to a value and can be used anywhere an expression is expected, including `let` bindings:

```oi
let label = match area(shape) {
  a if a > 100.0 => "large"
  a if a > 10.0  => "medium"
  _              => "small"
}
```

When matching an ADT variant but ignoring its fields, use `{ .. }`:

```oi
let name = match shape {
  Shape.Circle { .. }    => "Circle"
  Shape.Rectangle { .. } => "Rectangle"
  Shape.Triangle { .. }  => "Triangle"
}
```

## Lambdas and Closures
Anonymous functions use the `|args| body` syntax (Rust/Ruby style). Arrow functions (`=>`) are intentionally not used to prevent confusion with `match` arms.

```oi
-- Single-line lambda
let double = |x| x * 2

-- Multi-line closure with block
let process = |item| {
  log.info("Processing item")
  item.calculate()
}
```

## Pipeline Operator
The `|>` operator passes the result of the left-hand expression as the **first argument** of the right-hand function (Elixir-style).

```oi
-- "Hello" is passed as the first argument to String.repeat
let banner = "Hello"
  |> String.repeat(3)        -- String.repeat("Hello", 3)
  |> String.to_upper()       -- String.to_upper("HelloHelloHello")

-- Chaining data transformations
let results = List.range(1, 101)
  |> map(fizzbuzz)
  |> filter(|s| s != "")
```

### Standard Collection Functions

The following functions operate on collections and are called via the pipeline operator `|>` (see ADR-0010):

| Function | Description |
|----------|-------------|
| `map(f)` | Transform each element |
| `filter(f)` | Keep elements where `f` returns true |
| `each(f)` | Execute `f` for each element (for side effects) |
| `flat_map(f)` | Map and flatten one level |
| `reduce(init, f)` | Accumulate elements into a single value |
| `fold(init, f)` | Alias for `reduce` |

```oi
-- ✅ Canonical: pipeline style
List.range(1, 101)
  |> map(fizzbuzz)
  |> each(|s| io.println(s))

-- ✅ Single operation also uses pipeline
shapes |> each(|s| io.println(describe(s)))
```

### Pipeline `|>` vs Dot `.` Notation

Dot notation (`.`) is reserved for **methods defined on types** via trait implementations (e.g., `n.to_string()`, `list.length`). The pipeline `|>` is used for **standalone functions** that receive data as their first argument.

```oi
-- `.` = type-owned method
let s = n.to_string()

-- `|>` = standalone function
let doubled = numbers |> map(|x| x * 2)
```

> **Rule**: Collection operations (`map`, `filter`, `each`, etc.) are always invoked via `|>`, never via dot notation. This follows the "One Canonical Form" principle — there is exactly one way to express data transformation pipelines.

## Tests
Testing is a first-class citizen using `test` for unit testing and `property` for property-based testing.

```oi
test "사용자 생성 시 기본 역할은 Member" {
  let user = User.new(name: "Alice", email: "alice@test.com")
  assert user.role == Role.Member
}

property "세금은 항상 수입보다 작다" {
  forall income: Money where income > 0
  forall bracket: TaxBracket
  {
    let tax = calculate_tax(income, bracket)
    assert tax < income
  }
}
```

## Comments and Structured Metadata
- `--` is used for human-readable inline comments.
- `---` serves as YAML-style structured block metadata to be consumed by both AI agents and humans.

```oi
---
purpose: "사용자 인증을 처리하고 JWT 토큰을 발급한다"
domain: authentication
stability: stable
ai_notes: |
  이 함수를 수정할 때 rate_limiter 로직을 반드시 유지할 것.
  보안 감사 2025-03에 통과한 구현임.
---
fn authenticate(cred: Credentials) -> Result[Token, AuthError]
  with io, crypto
```

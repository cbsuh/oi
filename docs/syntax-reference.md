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

## Pattern Matching
Pattern matching replaces complex `if-else` chains. Exhaustive matching is required. You can add extra conditional logic using `if` guards.

```oi
match (value) {
  1 => "One"
  2 => "Two"
  x if x > 10 => "Greater than ten"
  _ => "Other"
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

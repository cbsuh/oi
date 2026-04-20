# ADR-0015: Loop Constructs

- Date: 2026-04-20
- Status: Accepted
- Participants: cbsuh, Claude Opus 4.6 (Thinking)

## Context

oi needs loop constructs for imperative iteration. The pipeline operator `|>` with `map`, `filter`, `each`, `reduce` is already the canonical form for collection operations (ADR-0010), and actors handle infinite event loops (ADR-0013). Loops serve as the imperative tool for cases where pipelines are awkward — primarily accumulation with `var` (ADR-0002) and conditional repetition.

The key design question is how many loop constructs to provide and whether to include `break`/`continue`.

## Options Considered

### Part 1: Loop Keywords

1. **`for` + `while`** — Two keywords with clear distinct roles.
2. **`for` only (Go-style)** — One keyword for all loop forms.
3. **`for` + `while` + `loop`** — Three constructs including dedicated infinite loop.

### Part 2: `break` / `continue`

1. **Both supported** — Full imperative loop control.
2. **`break` only** — Essential for `while true` exit.
3. **Neither** — No non-linear control flow in loops.

### Part 3: Range Syntax

1. **`..` operator** — `0..10` (exclusive), `0..=10` (inclusive).
2. **`List.range()` only** — Function-based.
3. **`to`/`until` keywords** — Kotlin-style.

## Decision

### Part 1: `for` + `while`
- `for x in collection { ... }` — iterate over a collection or range.
- `while condition { ... }` — repeat while condition is true.
- No `loop` keyword — `while true` is structurally unnecessary (see Part 2).

### Part 2: No `break`, no `continue`
Neither `break` nor `continue` are provided. All loop control flow is structured:

- **`for` loops always iterate the entire collection.** If early termination is needed, use pipeline operations (`find`, `take_while`, `any`).
- **`while` loops exit only when their condition becomes false.** Complex exit conditions are composed into the `while` condition using `var` flags.
- **Infinite event loops are handled by actors** (ADR-0013), not by `while true` + `break`.

### Part 3: `..` range operator
- `0..10` creates a `Range` (exclusive end, lazy).
- `0..=10` creates a `Range` (inclusive end, lazy).
- `List.range()` remains for creating concrete `List` values in pipelines.

### Additional: Loops are statements
Loops always return Unit `()`. They are not expressions. This differs from `if`/`match` (which are expressions) because loops may execute zero times, making a return value unreliable.

## Rationale

### No break/continue

This is the most significant decision. The rationale follows from oi's existing design:

**Every use case for `break` has a better alternative in oi:**

| Pattern | With `break` | oi alternative |
|---------|-------------|----------------|
| Find first match | `for x in list { if match { break } }` | `list \|> find(pred)` |
| Take until condition | `for x in list { if done { break } }` | `list \|> take_while(pred)` |
| Conditional early exit | `while true { if done { break } }` | `while !done { ... }` |
| Infinite event loop | `while true { msg = recv(); break on quit }` | `actor` with `on` handlers |

**Every use case for `continue` has a better alternative:**

| Pattern | With `continue` | oi alternative |
|---------|----------------|----------------|
| Skip invalid items | `for x in list { if invalid { continue }; process(x) }` | `list \|> filter(valid) \|> each(process)` |
| Skip in loop body | `for x in list { if skip { continue }; ... }` | `for x in list { if !skip { ... } }` |

**Design principle alignment:**

- **One Canonical Form**: Without `break`/`continue`, there is exactly one way to exit each loop type — `for` exhausts its collection, `while` exits when its condition is false. No alternatives.
- **AI generation friendliness**: AI can reason about loop termination trivially — `for` always terminates, `while` terminates when its condition evaluates to false. No hidden exit points.
- **Human review friendliness**: A reviewer sees `for x in items { ... }` and *knows* every item will be processed. No need to scan the body for `break` statements. Similarly, `while cond { ... }` exits when `cond` is false — the exit condition is always in the header.
- **Effect Transparency**: Loop behavior is fully determined by its header. No "invisible control flow" inside the body.

**The `while true` question:**

Without `break`, `while true { ... }` is an infinite loop that can never terminate. Since actors handle all infinite processing patterns (ADR-0013), there is no legitimate use for `while true` in oi. The compiler should warn on `while true` (or any statically-detectable non-terminating `while` condition).

### for + while (not Go-style for-only)

- `for condition { ... }` (Go's while-equivalent) is syntactically confusing — `for` implies iteration over something.
- `while` communicates "repeat until condition changes" clearly.
- Two keywords with zero overlap: `for` = iterate, `while` = conditional repeat.

### No `loop` keyword

- `loop` would exist solely for `while true` + `break`. Since we have neither `break` nor `while true`, `loop` has no purpose.

### Range syntax `..`

- `for i in 0..10` is concise and widely recognized (Rust, Ruby, Kotlin).
- `for i in List.range(0, 10)` is verbose and creates an unnecessary concrete list.
- `Range` is a distinct lazy type from `List`, so having both `..` and `List.range()` does not violate "One Canonical Form" — they serve different purposes.

### Loops as statements (not expressions)

- `for` may iterate over an empty collection → no value produced.
- `while` condition may be false initially → body never executes.
- Making loops expressions would require `break value` (Rust's approach), but we intentionally excluded `break`.
- `if`/`match` are expressions because exactly one branch always executes. Loops lack this guarantee.

## Consequences

### Positive
- **Zero non-linear control flow** in loops. All jumps (`break`, `continue`) eliminated.
- **Predictable loop behavior**: `for` = full iteration, `while` = condition-driven exit. No exceptions.
- **Reinforces pipeline-first design**: Users naturally reach for `|> filter()`, `|> find()`, `|> take_while()` instead of `for` + `break`.
- **Simpler compiler**: No `break`/`continue` target resolution needed.
- **Two fewer keywords**: `break` and `continue` are not reserved.

### Trade-offs
- **Some patterns become more verbose**: Conditional repetition with multiple exit conditions requires `var` flags composed into the `while` condition.
- **Uncommon choice**: Very few languages omit `break`/`continue`. Developers from C/Rust/Python backgrounds may find this surprising.
- **`while true` is effectively useless**: The compiler should detect and warn on this pattern since it can never terminate.
- **Pipeline dependency**: Early termination from collections requires pipeline functions (`find`, `take_while`) to be available in the standard library from Phase 2.

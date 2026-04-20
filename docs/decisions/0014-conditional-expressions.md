# ADR-0014: Conditional Expressions and Control Flow Parentheses

- Date: 2026-04-20
- Status: Accepted
- Participants: cbsuh, Claude Opus 4.6 (Thinking)

## Context

oi's syntax reference defines `match` and `for` as control flow constructs but has no `if`/`else` conditional. Every general-purpose language provides some form of conditional branching beyond pattern matching. Without `if`/`else`, simple Boolean checks like `if age >= 18` must be expressed as `match (cond) { true => ..., false => ... }`, which violates "Explicit but Concise."

Additionally, the existing documentation is inconsistent about parentheses in control flow keywords: `match (value)` uses parentheses while `for x in list` does not.

## Options Considered

### Part 1: Conditional Expressions

1. **Introduce `if`/`else` as a standalone construct** — A new expression form independent of `match`.
2. **No `if`/`else` (match only)** — All Boolean branching done via `match` on `true`/`false`.
3. **`if`/`else` as syntactic sugar for `match`** — User writes `if`/`else`, AST represents it as `match` on `Bool`.

### Part 2: Control Flow Parentheses

1. **All parentheses** — `if (cond)`, `match (value)`, `for (x in list)`.
2. **No parentheses** — `if cond`, `match value`, `for x in list`.
3. **Mixed** — `match (value)` with parentheses, `if cond` and `for x in list` without.

### Part 3: Related Features

1. **`if` as expression** (returns a value) vs **`if` as statement** (no return value).
2. **Ternary operator** (`cond ? a : b`) — introduce or not.
3. **`if let`** pattern — introduce or defer.
4. **`else if` vs `elif`** — keyword form.

## Decision

### Part 1: Option 3 — `if`/`else` as syntactic sugar for `match`
- Users write `if`/`else`; the AST may desugar to `match` on `Bool`.
- From the user's perspective, `if`/`else` is a first-class construct.

### Part 2: Option 2 — No parentheses for all control flow keywords
- `if cond { ... }`, `match value { ... }`, `for x in list { ... }`.
- Parentheses are allowed syntactically (as expression grouping) but `oi-fmt` removes unnecessary outermost parentheses from control flow conditions/subjects.

### Part 3:
- **`if` is an expression** — it returns a value and can be used in `let` bindings.
- **No ternary operator** — `if`/`else` expression makes it redundant.
- **No `if let`** — deferred to Phase 2 evaluation. `match` handles all pattern matching.
- **`else if`** (two keywords) — not `elif`.
- **Braces always required** — no single-expression shorthand.

## Rationale

### if/else introduction
- "Explicit but Concise": `match true/false` for Boolean checks is verbose and obscures intent.
- AI generation: Every AI model is trained extensively on `if`/`else` patterns.
- Human review: `if age >= 18` is universally understood; `match (age >= 18) { true => ... }` requires an extra mental step.
- Syntactic sugar preserves "One Canonical Form" at the AST level while providing concise surface syntax.

### No parentheses
- "One Canonical Form": One rule — "control flow keywords are never followed by required parentheses." No exceptions.
- Parentheses are unnecessary for parsing: `{` unambiguously marks the start of the body in all cases.
- Consistency: `for x in list` already uses no parentheses. Extending this to `match` and `if` unifies the style.
- Precedent: Rust, Go, Swift — all modern languages designed from scratch have dropped mandatory control flow parentheses.
- Expression grouping parentheses remain valid (e.g., `if (a > 1) || (a == -1) { ... }`), but `oi-fmt` strips unnecessary outermost parentheses (e.g., `if (x > 0) { ... }` → `if x > 0 { ... }`).

### Expression semantics
- `match` is already an expression (ADR-0007). Making `if` also an expression maintains consistency.
- Eliminates the need for a ternary operator (`?:`), which would violate "One Canonical Form."

### else if over elif
- `else if` is used by the vast majority of languages AI models are trained on (Rust, Go, C, Java, JS, Kotlin, Swift).
- `elif` would introduce a new keyword for negligible conciseness gain.

### Braces always required
- "One Canonical Form": Exactly one way to write an `if` block.
- Consistent with all other block constructs in oi (`fn`, `match`, `for`, `type`).

### if let deferred
- `match` already handles all pattern matching needs.
- Introducing `if let` adds a second pattern matching construct, conflicting with "One Canonical Form."
- Can be reconsidered in Phase 2 if `match` on `Option` proves too verbose in practice.

## Consequences

### Positive
- Simple Boolean branching is now concise and natural.
- All control flow keywords follow a single, consistent parenthesization rule.
- No ternary operator needed — fewer constructs to learn.
- `oi-fmt` can enforce canonical formatting for `if` conditions.

### Trade-offs
- `if`/`else` and `match` are two ways to branch, though they serve different roles (Boolean vs pattern). The AST-level unification mitigates this.
- Removing parentheses from `match` changes existing documented examples (one-time documentation update).
- Without `if let`, `Option` handling always requires a full `match` with both branches.

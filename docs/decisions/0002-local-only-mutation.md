# ADR-0002: Local-Only Mutation with `var`

- Date: 2026-04-05
- Status: Accepted
- Participants: cbsuh, AI (Claude Opus 4.6)

## Context
The language needed a mutability model. The syntax reference documented `let` as immutable but provided no mechanism for mutable bindings. Complex accumulation patterns (counters, batch processing) require some form of state update.

## Options Considered
1. **`let mut x = 10`** — Rust style. Mutability is a modifier on `let`.
2. **`var x = 10`** — Swift/Kotlin style. `let` and `var` are distinct keywords.
3. **No mutable variables** — Pure functional style. All state changes via recursion, `fold`, or effect system.

## Decision
**`var` keyword with Local-Only Mutation: `var` is allowed only inside function bodies and cannot cross function boundaries.**

Specifically:
- ✅ `var` inside a function body (local accumulators, loop counters)
- ❌ `var` as a function parameter
- ❌ `var` at module level (no global mutable state)
- ❌ `var` in type/struct field definitions

## Rationale
- **One Canonical Form**: `let` always means immutable, `var` always means mutable — two distinct keywords with zero ambiguity. `let mut` uses the same keyword (`let`) for two different behaviors, which is less canonical.
- **Effect Transparency preserved**: Since `var` cannot escape a function, the function signature remains pure from an external perspective. No `with` annotation is needed for local mutation. AI and humans can trust the signature.
- **Self-Verifiable Code**: Contract verification (`requires`/`ensures`) is dramatically simpler when variables cannot mutate across function boundaries. The verifier only needs to track local state within one function scope.
- **Practical conciseness**: Pure functional style (option 3) was considered but rejected because oi explicitly discarded monad transformers. Forcing all state into `fold`/`reduce` would re-introduce the functional complexity that oi aims to avoid, and would make complex business logic harder for humans to review.
- **AI generation friendly**: `var` as a distinct token is unambiguous for AI models. A modifier pattern (`let mut`) is more prone to accidental omission during code generation.

## Consequences
- oi achieves "pure on the outside, practical on the inside" — functions look pure from their signatures but can use imperative patterns internally.
- `fold`/`reduce` remains available as an alternative style. Neither is forced.
- Data structures are always immutable. Record updates require functional update syntax (e.g., `user.with(name: "new")`).
- Global mutable state is impossible without the effect system (`with`), ensuring all shared state is tracked.

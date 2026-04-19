# ADR-0011: No Built-in `type_name()` Method

- Date: 2026-04-19
- Status: Accepted
- Participants: cbsuh, Antigravity (Claude Opus 4.6)

## Context
The `pattern_matching.oi` example used `shape.type_name()` to get the runtime name of an ADT variant as a string. However, this method was undocumented — no trait, no built-in specification, and no standard library entry defined it. This raised the question: should oi provide a built-in way to get a type's name at runtime?

## Options Considered
1. **Built-in on all types** — Compiler automatically provides `.type_name()` on every type, like Rust's `std::any::type_name`.
2. **Explicit `Named` trait** — Define a `trait Named { fn type_name(self) -> String }` that types opt into via `deriving` or manual implementation.
3. **No built-in; use pattern matching** — Do not provide `.type_name()` at this stage. Use `match` to explicitly map variants to strings when needed.

## Decision
**Option 3: No built-in `type_name()`. Use pattern matching instead.**

## Rationale
- **"Explicit but Concise"**: A magic `.type_name()` available on all types is implicit behavior — the user cannot see where it comes from. Pattern matching makes the logic explicit and visible.
- **Design phase caution**: oi has no standard library or reflection system yet. Committing to a reflection primitive now would constrain future design decisions around metaprogramming and runtime type information.
- **Better example**: Replacing `.type_name()` with a `match` expression in the example actually demonstrates more oi features (match as expression, exhaustive ADT handling) and is more pedagogically valuable.
- **Future-compatible**: When oi designs its reflection/metaprogramming system (likely Phase 2+), a `type_name`-like capability can be introduced with proper trait-based design (Option 2). This decision does not prevent that.

## Consequences
- `pattern_matching.oi` is updated to use `match` instead of `.type_name()`.
- No reflection primitives exist in the language at this stage.
- A future ADR may introduce a `Named` or `Reflectable` trait when the standard library is designed.

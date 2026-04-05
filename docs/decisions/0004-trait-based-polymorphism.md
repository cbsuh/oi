# ADR-0004: Trait-Based Polymorphism with Inline Implementation

- Date: 2026-04-05
- Status: Accepted
- Participants: cbsuh, AI (Claude Opus 4.6), AI (Gemini 3.1 Pro — identified the gap)

## Context
The language had no mechanism for polymorphism or interfaces. Without one, AI models would hallucinate syntax from other languages (`interface`, `protocol`, `abstract class`, etc.) when asked to write generic or polymorphic code.

## Options Considered
1. **`trait` + separate `impl` block** — Rust style. Traits defined separately, implemented in standalone `impl` blocks.
2. **`interface` + structural matching** — Go style. Types satisfy an interface implicitly if they have matching methods.
3. **`trait` + inline `with` implementation** — Rust variant. Traits can be implemented inline at the type definition site using `with Trait { ... }`, with standalone `impl` blocks available for external types.

## Decision
**Option 3: `trait` with inline `with` implementation, plus standalone `impl` for external types.**

## Rationale
- **Explicit but Concise**: Option 2 (structural matching) was ruled out because oi's `comparison.md` explicitly discards "Implicit Typeclass Resolution" — behavior depending on what methods happen to exist is implicit magic. Traits require explicit opt-in.
- **AI generation friendliness**: With inline implementation, an AI sees the type definition and all its trait implementations in one place. No need to search across files for scattered `impl` blocks. This centralizes context, which is critical for LLM context windows.
- **One Canonical Form**: While two syntax forms exist (inline `with` and standalone `impl`), they serve distinct, non-overlapping purposes:
  - `with` at type definition: for traits you implement when defining your own type.
  - `impl` block: for implementing traits on types you don't own (e.g., library types).
- **Explicit over implicit**: Every trait implementation is a conscious, visible declaration — not an accident of having the right method names.

## Consequences
- `trait` defines a set of required methods.
- Types implement traits inline using `with Trait { ... }` at definition time.
- For external types, standalone `impl Trait for Type { ... }` blocks are used.
- Generic constraints use `[T: Trait]` syntax (square brackets per ADR-0001).
- Orphan rules may be needed in the future to prevent conflicting implementations.

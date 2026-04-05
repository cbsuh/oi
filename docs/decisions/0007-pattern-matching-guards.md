# ADR-0007: Pattern Matching Guards

- Date: 2026-04-05
- Status: Accepted
- Participants: cbsuh, AI (Gemini 3.1 Pro)

## Context
Example code (`examples/pattern_matching.oi`) used conditional rules inside pattern matching (e.g., `a if a > 100.0 =>`). However, "Guards" were not officially documented in the syntax specification, leading to potential AI hallucinations (e.g., imagining a `when` keyword).

## Options Considered
1. **Option A: `pattern if condition =>`** (Rust, Scala style)
2. **Option B: `pattern when condition =>`** (F#, C# style)
3. **Option C: No guard syntax** — Force developers to write standard `if/else` logic inside the match arm.

## Decision
**Option A: `pattern if condition =>`**

## Rationale
- **Minimalism (One Canonical Form)**: `if` is already a core reserved word in `oi` for boolean conditions. Reusing it for guards avoids adding a new reserved keyword like `when` to the language definition.
- **AI familiarity**: The `if` guard pattern is extremely common (Rust/Scala/Python list comprehensions) and models generate it naturally.
- **Consistency**: It aligns directly with the existing code in `examples/pattern_matching.oi` without requiring backward modifications.

## Consequences
- Guards are officially supported in `match` statements utilizing the `if` keyword immediately following a pattern binding.
- Validated via `syntax-reference.md`.

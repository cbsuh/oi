# ADR-0001: Use Square Brackets for Generics

- Date: 2026-04-05
- Status: Accepted
- Participants: cbsuh, AI (Claude Opus 4.6)

## Context
Project documentation used two different notations for generics inconsistently: angle brackets `Result<T, E>` (Rust/TypeScript style) and square brackets `Result[T, E]` (Scala/Kotlin style). This inconsistency confused readers and needed to be resolved into a single canonical form.

## Options Considered
1. **`<T, E>` (angle brackets)** — Familiar to Rust, TypeScript, Java, C# developers. Consistent with oi's heavy borrowing from Rust.
2. **`[T, E]` (square brackets)** — Used by Scala, Kotlin. No collision with comparison operators or HTML/XML markup.

## Decision
**Square brackets `[T, E]`.**

## Rationale
- oi is designed for AI-human collaboration. AI communicates code through Markdown, chat, and HTML — all contexts where `<` and `>` are interpreted as markup and can cause code to silently disappear.
- During this project's own development, a mixed notation error (`Result<Response, HttpError]`) occurred, demonstrating the practical risk of angle brackets.
- Square brackets eliminate the C++ `>>` ambiguity problem (bitshift vs template closing), aligning with oi's "One Canonical Form" principle — generics and comparison operators are completely distinct tokens.
- While Rust developers may find it unfamiliar initially, the choice prioritizes oi's core audience: AI agents and the humans who review AI-generated code in mixed-media environments.

## Consequences
- All generics across the language use `[T, E]`: `Result[T, E]`, `List[T]`, `Map[K, V]`.
- All existing documentation and examples have been updated to use square brackets.
- Developers coming from Rust/TypeScript will need a brief adjustment period.

# ADR-0008: Collection Indexing

- Date: 2026-04-05
- Status: Accepted
- Participants: cbsuh, AI (Gemini 3.1 Pro)

## Context
Standard bracket notation (`list[index]`) was being used in examples, but ADR-0001 had already reserved the bracket notation `[]` for Generics (e.g., `Result[T, E]`). This generated a potential syntactic ambiguity: does `foo[bar]` mean type parameterization or index access?

## Options Considered
1. **Option A: `list[index]`** — The classic 0-based syntax (C/Python/JS/Rust).
2. **Option B: `list.at(index)`** — Strictly method-based.
3. **Option C: `list(index)`** — Scala style (uses parenthesis).

## Decision
**Option A: `list[index]`**

## Rationale
- **Disambiguation via Capitalization**: `oi` strictly enforces that User-Defined Types must begin with an uppercase letter, and variables/values must begin with a lowercase letter. As a result, the parser and the reader can trivially distinguish `Result[Int]` (generics over Types) from `list[0]` or `items[index]` (indexing via values).
- **Familiarity**: Array bracket indexing is a deeply ingrained idiom. Forcing developers to write `.at(0)` violates the conciseness goal for no practical safety gain. 

## Consequences
- Bracket indexing `[index]` is officially the standard way to access collections.
- The `syntax-reference.md` has been updated to reflect this.

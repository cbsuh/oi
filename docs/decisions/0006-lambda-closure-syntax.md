# ADR-0006: Lambda and Closure Syntax

- Date: 2026-04-05
- Status: Accepted
- Participants: cbsuh, AI (Gemini 3.1 Pro)

## Context
Anonymous functions (lambdas/closures) were being used in example code (e.g., `list \|> map(\|x\| x * 2)`) but lacked formal definition in the specification. A canonical syntax is required to prevent AI from hallucinating `=>` or `fn() {}` styles interchangeably.

## Options Considered
1. **Option A: `\|args\| body`** — Rust / Ruby style.
2. **Option B: `(args) => body`** — JavaScript / TypeScript / C# style.
3. **Option C: `fn(args) { body }`** — Go style.

## Decision
**Option A: `\|args\| body`**

## Rationale
- **One Canonical Form & Disambiguation**: The `=>` operator is already utilized extensively in `oi` for `match` arms and `ensures` contracts (e.g., `ensures result => ...`). Reusing it for lambdas would create visual ambiguity for both the parser and human reviewers.
- **Visual Harmony**: The pipe character `\|` visually synergizes with the pipeline operator `\|>`, resulting in clean, readable chains: `items \|> filter(\|x\| x > 0) \|> map(\|x\| x * 2)`.
- **Explicit but Concise**: It is the most concise syntax available, stripping away unnecessary parentheses or `fn` keywords for short operations.
- **Consistency**: It aligns with the existing examples produced during earlier design phases.

## Consequences
- Arrow functions `=>` are explicitly invalid for lambdas in `oi`.
- Multi-line closures can wrap the body in curly braces: `\|x\| { ... }`.
- The `syntax-reference.md` has been updated to include this formal definition.

# ADR-0005: String Interpolation Syntax

- Date: 2026-04-05
- Status: Accepted
- Participants: cbsuh, AI (Gemini 3.1 Pro)

## Context
String interpolation was used in example files (`examples/pattern_matching.oi`) but was not formally defined in the language specification. AI models require a definitive rule to avoid hallucinating other languages' syntax like `${var}` or `f"{var}"`.

## Options Considered
1. **Option A: `{var}`** — Direct interpolation without prefixes (Rust, Python f-strings, C#).
2. **Option B: `${var}`** — Explicit indicator (TypeScript, Kotlin).
3. **Option C: `\(var)`** — Swift style.
4. **Option D: No interpolation** — Enforce explicit `append` or string formatting functions (Go).

## Decision
**Option A: `{var}`**

## Rationale
- **Explicit but Concise**: `{var}` is the most concise way to interpolate variables while remaining visually distinct from regular text. Requiring a prefix character (`$` or `f`) introduces unnecessary visual noise that `oi` seeks to minimize.
- **Consistent with existing examples**: It directly validates the intuitive code generated during earlier design phases without requiring rewrites.
- **AI generation friendliness**: The `{var}` pattern is heavily represented in LLM training data, meaning models will correctly generate it without requiring complex prompting.

## Consequences
- Strings in `oi` support direct variable interpolation using `{...}` syntax.
- If literal curly braces are needed in a string, they must be escaped (e.g., `\{` and `\}`).
- The `syntax-reference.md` has been updated to officially document this feature.

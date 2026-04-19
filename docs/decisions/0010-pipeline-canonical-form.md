# ADR-0010: Pipeline as Canonical Form for Collection Operations

- Date: 2026-04-19
- Status: Accepted
- Participants: cbsuh, Antigravity (Claude)

## Context

Two different call styles were found across the codebase for collection operations like `each`, `map`, and `filter`:

- **Pipeline style**: `list |> each(f)` — used in `README.md`
- **Method style**: `list.each(f)` — used in `fizzbuzz.oi`, `pattern_matching.oi`

Both styles express the same semantics, but having two valid ways to write identical logic violates oi's **"One Canonical Form"** principle. Additionally, the method style raises an undocumented question: does oi support UFCS (Uniform Function Call Syntax)?

## Options Considered

1. **Option A: Pipeline `|>` as canonical form** — `each`, `map`, `filter` etc. are standalone functions invoked via `|>`. Method-call syntax (`.each(...)`) is not valid for these operations.
2. **Option B: Method call as canonical form** — `.each(...)`, `.map(...)` are methods on collection types. Pipeline `|>` is not used with these functions.
3. **Option C: Both are valid** — Document UFCS as a language feature, making `list |> f(...)` and `list.f(...)` interchangeable.

## Decision

**Option A: Pipeline `|>` as the canonical form for collection operations.**

Collection operations (`map`, `filter`, `each`, `flat_map`, `reduce`, `fold`, etc.) are **standalone functions** that receive a collection as their first argument. They are called via the pipeline operator `|>`.

```oi
-- ✅ Canonical form
List.range(1, 101)
  |> map(fizzbuzz)
  |> each(|s| io.println(s))

-- ❌ Not valid
results.each(|s| io.println(s))
results.map(fizzbuzz)
```

For single invocations without chaining, the pipeline is still used:

```oi
-- ✅ Canonical form (single operation)
shapes |> each(|s| io.println(describe(s)))
```

## Rationale

1. **"One Canonical Form"**: Only one way to call these operations. No UFCS ambiguity.
2. **Pipeline consistency**: `|>` is oi's primary composition mechanism. Mixing it with `.method()` mid-chain breaks the visual flow and creates two patterns for readers to track.
3. **Simplicity**: No need to introduce UFCS — a complex feature that would allow `f(x, y)` ↔ `x.f(y)` conversion and multiply the number of valid forms for any expression.
4. **Method syntax reserved for true methods**: Dot notation (`.`) is reserved for methods defined on types via `trait` implementations (e.g., `n.to_string()`, `list.length`). This makes a clear semantic distinction: `.` means "this type owns this method", `|>` means "pass this value to a function".

## Consequences

### Positive
- Clear separation: `.method()` = type-owned behavior, `|> function()` = data transformation pipeline.
- No UFCS complexity. The language remains simpler.
- All collection operation chains read top-to-bottom uniformly.

### Trade-offs
- Single-operation calls like `shapes |> each(f)` may feel slightly verbose compared to `shapes.each(f)` for developers from OOP backgrounds.
- Developers must remember which operations are standalone functions (pipeline) vs. type methods (dot notation).

### Required Changes
- Update `examples/fizzbuzz.oi`: `results.each(...)` → pipeline style
- Update `examples/pattern_matching.oi`: `shapes.each(...)` → pipeline style
- Update `docs/syntax-reference.md`: Document `each` as a standard function and add canonical form guidance to Pipeline Operator section

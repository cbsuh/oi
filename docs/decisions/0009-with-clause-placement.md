# ADR-0009: `with` Clause Placement

- Date: 2026-04-19
- Status: Accepted
- Participants: Human (cbsuh), AI (Claude)

## Context
The `with` keyword declares side effects on functions. Across documentation and examples, two placement styles were used inconsistently:

**Style A — Inline (same line as signature):**
```oi
fn save_user(user: User) -> Result[UserId, DbError] with db {
```

**Style B — Next line (indented below signature):**
```oi
fn get_user(uid: UserId) -> Result[User, AppError]
  with io, db
{
```

This violated oi's "One Canonical Form" principle: there should be exactly one way to express any given meaning. The inconsistency appeared across `syntax-reference.md` (inline style), `effect-system.md` (both styles), `user_service.oi` (next-line style), and other examples (inline style).

## Options Considered
1. **Always inline** — `with` on the same line as the function signature.
2. **Always next line** — `with` on a dedicated line below the signature, indented by 2 spaces.
3. **Conditional** — Inline when short (single effect, no contracts), next line otherwise.

## Decision
**Option 2: Always next line.** The `with` clause is always placed on its own line, indented by 2 spaces below the function signature.

```oi
fn print_message(msg: String)
  with io
{
  io.println(msg)
}

fn get_user(uid: UserId) -> Result[User, AppError]
  with io, db, log
{
  ...
}

pub fn find_user(id: UserId) -> Result[User, UserError]
  with io, db
  requires id.is_valid()
  ensures result => result.is_ok()
{
  ...
}
```

Pure functions (no `with` clause) remain unchanged:
```oi
fn add(a: Int, b: Int) -> Int {
  a + b
}
```

## Rationale

1. **One Canonical Form**: Eliminates the need for case-by-case formatting judgment. There is exactly one valid placement for `with`, regardless of how many effects or other clauses exist.

2. **Consistency with `requires`/`ensures`**: Contract clauses already use the next-line style. Having all function signature clauses (`with`, `requires`, `ensures`) follow the same formatting rule creates a predictable, uniform structure:
   - Line 1: `fn name(params) -> ReturnType`
   - Line 2: `with effects`
   - Line 3: `requires precondition`
   - Line 4: `ensures postcondition`

3. **Formatter-aligned culture**: Modern developers are accustomed to opinionated formatters (`gofmt`, `rustfmt`, `prettier`). A single canonical placement removes formatting debates entirely — the format *is* the spec.

4. **AI-friendly line-level parsing**: Each line carries one semantic unit, making it easier for AI models to parse and generate function signatures.

## Consequences

**Positive:**
- Zero ambiguity in function signature formatting.
- All documentation and examples become consistent.
- AI code generation requires no conditional formatting logic.
- Aligns with the broader oi principle that formatting is part of the language spec.

**Trade-off:**
- Simple effectful functions like `fn main() with io { ... }` become more verbose (3+ lines instead of 1). This is an accepted cost for consistency and predictability.

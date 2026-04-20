# ADR-0018: Effect System Semantics

- Date: 2026-04-20
- Status: Accepted
- Participants: cbsuh, Antigravity (Claude Opus 4.6)

## Context
The `with` keyword for declaring side effects was established early in oi's design (ADR-0009 placement rules, ADR-0013 concurrency effects). However, four semantic questions remained unresolved: (1) how effects propagate from callee to caller, (2) whether a `without` restriction mechanism is needed, (3) whether effects form a hierarchy, and (4) how custom effects are defined.

## Decisions

### 1. Effect Propagation: Manual Declaration + Compiler Verification

**Options Considered:**
1. **Automatic inference** — Callee effects automatically propagate to caller; `with` can be omitted.
2. **Manual declaration** — Caller must list all effects in its `with` clause.
3. **Manual declaration + compiler verification** — Caller must list all effects; compiler errors on omission.

**Decision: Option 3 (manual + compiler verification).**

Rules:
- Every function must declare **all** effects it uses, either directly or transitively through callees.
- If a function calls another function that requires an effect not listed in the caller's `with` clause, the compiler emits an error.
- The error message must name the callee and the missing effect.

```oi
fn save_user(user: User) -> Result[Unit, DbError]
  with db
{
  db.insert(user)
}

fn register(name: String) -> Result[Unit, DbError]
  with db              -- Required: save_user needs `db`
{
  let user = User.new(name)
  save_user(user)?
}

-- If `with db` is omitted from register():
-- ERROR: register() calls save_user() which requires `db` effect
```

**Rationale:**
- **Effect Transparency**: This is oi's core principle #5. A function's signature must reveal every side effect. Automatic inference would hide effects, directly violating this principle.
- **AI friendliness**: AI generates the `with` clause explicitly. If it forgets one, the compiler catches it — creating a feedback loop for self-correction.
- **Human review**: Reviewers see all effects at a glance without tracing call chains.

---

### 2. No `without` Mechanism (Phase 1)

**Options Considered:**
1. **No `without`** — Pure functions are those without a `with` clause. No explicit restriction syntax.
2. **`without` keyword** — Explicitly forbid certain effects in a scope.
3. **Generic constraints** — Express effect restrictions via type-level constraints.

**Decision: Option 1 (no `without` in Phase 1).**

**Rationale:**
- **One Canonical Form**: Purity is already expressed by the absence of `with`. Adding `without` creates a second way to express "this is pure."
- **Practical sufficiency**: If a function has no `with`, it is guaranteed pure. If a higher-order function accepts a callback, the callback's effect signature is part of its type — the compiler already enforces it.
- **Deferred**: If real-world usage reveals a need for `without` (e.g., sandboxing), it can be introduced in Phase 2+ without breaking changes.

```oi
-- Pure function: no `with` = guaranteed pure
fn double(x: Int) -> Int {
  x * 2
}

-- Higher-order: callback type determines allowed effects
fn apply_pure(f: fn(Int) -> Int, x: Int) -> Int {
  f(x)    -- f has no effects in its type → guaranteed pure
}
```

---

### 3. Effect Hierarchy: Fine-Grained + Convenience Groups

**Options Considered:**
1. **Flat** — All effects are independent and equal.
2. **Hierarchical** — Some effects are subsets of others (`io` contains `network`).
3. **Fine-grained + convenience groups** — Individual effects are primary; `io` is an alias for a group.

**Decision: Option 3 (fine-grained + convenience groups).**

Standard effect hierarchy:
```
io (convenience group)
├── network    -- HTTP, TCP, UDP, DNS
├── fs         -- File system read/write
├── stdout     -- Standard output
└── stderr     -- Standard error

db             -- Database access (independent)
log            -- Logging/telemetry (independent)
crypto         -- Cryptographic operations (independent)
spawn          -- Actor creation (independent, ADR-0013)
concurrent     -- Parallel execution (independent, ADR-0013)
```

Rules:
- `with io` is equivalent to `with network, fs, stdout, stderr`.
- A function declared `with io` satisfies callees that require `network`, `fs`, `stdout`, or `stderr`.
- A function declared `with network` does **not** satisfy callees that require `fs`.
- In production code, prefer fine-grained effects. Use `io` for scripts, `main()`, and prototyping.

```oi
-- Fine-grained (production)
fn fetch(url: String) -> Result[String, HttpError]
  with network
{ ... }

-- Convenience group (scripts/main)
fn main()
  with io                    -- io ⊃ network, fs, stdout, stderr
{
  let data = fetch("https://api.example.com")?   -- ✅ io includes network
  io.println(data)
}

-- Compiler enforces granularity
fn process() -> Result[Unit, Error]
  with db
{
  fetch("...")?              -- ❌ ERROR: missing `network` effect
}
```

**Rationale:**
- **Effect Transparency**: Fine-grained effects reveal exactly what a function touches (`network` vs `fs`). This precision helps AI reason about function behavior and helps humans assess security/performance impact.
- **Explicit but Concise**: `io` as a convenience group prevents verbosity in entry points and scripts. The granularity is available when needed, but not forced everywhere.
- **One Canonical Form**: `io` is a well-defined alias, not an alternative system. The compiler can expand it predictably.

---

### 4. Custom Effects: Deferred to Phase 2+

**Options Considered:**
1. **Reuse `trait`** — Custom effects are defined as traits, used with `with TraitName`.
2. **`effect` keyword** — A dedicated keyword for defining effects.
3. **Defer** — Finalize only the standard effect list now; custom effect syntax later.

**Decision: Option 3 (defer to Phase 2+).**

The standard effects for Phase 1 are:
- `io` (group: `network`, `fs`, `stdout`, `stderr`)
- `db`
- `log`
- `crypto`
- `spawn` (ADR-0013)
- `concurrent` (ADR-0013)

**Rationale:**
- **Design phase caution**: The choice between `trait` and `effect` keyword has deep implications for effect handlers, effect polymorphism, and composition. These require implementation experience to evaluate properly.
- **Standard effects are sufficient**: The 10 standard effects (6 independent + 4 in the `io` group) cover the vast majority of Phase 2 use cases.
- **Future-compatible**: Both `trait`-based and `effect`-keyword approaches remain viable. This decision does not constrain Phase 2.

## Consequences

### Positive
- Effect signatures are always complete and verifiable — the compiler enforces correctness.
- Fine-grained effects enable precise security and performance reasoning.
- The `io` convenience group keeps entry points concise.
- No premature commitment to custom effect syntax.

### Trade-offs
- Manual declaration is more verbose than automatic inference. Every transitive effect must be listed. This is intentional — visibility is the design goal.
- No `without` means effect restriction on callbacks relies solely on type signatures, which is sufficient but may feel indirect.
- Custom effect deferral means Phase 2 libraries cannot define domain-specific effects until the syntax is decided.

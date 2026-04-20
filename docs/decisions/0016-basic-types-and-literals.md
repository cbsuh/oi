# ADR-0016: Basic Types and Literals

- Date: 2026-04-20
- Status: Accepted
- Participants: cbsuh, Antigravity (Claude Opus 4.6)

## Context
Before implementing the Phase 2 parser, oi needs a definitive specification for its primitive types, numeric literals, character handling, collection literals, and the unit type. These decisions affect every layer of the language — from lexer tokens to type inference rules. The TODO list identified five sub-items under "기본 타입과 리터럴" that required resolution.

## Decisions

### 1. Numeric Types: `Int` (64-bit) + Fixed-Width Opt-in

**Options Considered:**
1. **Single `Int` (64-bit only)** — No fixed-width types. Simple but insufficient for FFI/performance work.
2. **`Int` (64-bit) + fixed-width opt-in** — Default `Int` = `Int64`, with `Int8`/`Int16`/`Int32`/`UInt8`/`UInt16`/`UInt32`/`UInt64` and `Float32`/`Float64` available when needed.
3. **Arbitrary-precision `Int` + fixed-width** — `Int` = BigInt. Performance unpredictable; the runtime cost is invisible to the programmer.

**Decision: Option 2.**

The default types are:
- `Int` — 64-bit signed integer (alias for `Int64`)
- `Float` — 64-bit IEEE 754 floating point (alias for `Float64`)
- `Bool` — `true` or `false`
- `String` — UTF-8 encoded text

Fixed-width numeric types are available for opt-in:
- Signed: `Int8`, `Int16`, `Int32`, `Int64`
- Unsigned: `UInt8`, `UInt16`, `UInt32`, `UInt64`
- Floating: `Float32`, `Float64`

**Rationale:**
- **One Canonical Form**: AI defaults to `Int` and `Float` — no decision paralysis. Fixed-width types are only used when precision matters (FFI, binary protocols, game servers).
- **Explicit but Concise**: The default covers 95% of use cases. When you need `UInt8`, you say so explicitly — no implicit truncation or widening.
- **Target market alignment**: Game server development and Protobuf/FlatBuffers integration (see target-market-notes.md) require fixed-width integers.

---

### 2. Literal Syntax: Full Set Without Suffixes

**Options Considered:**
1. **Full set** — `0x` (hex), `0b` (binary), `0o` (octal), `_` separator, `e` scientific notation.
2. **Minimal set** — Only `_` separator; defer hex/binary to Phase 2+.
3. **Full set + type suffixes** — Add `42u8`, `3.14f32` style suffixes.

**Decision: Option 1 (full set, no suffixes).**

Supported literal forms:
| Literal | Example | Type |
|---------|---------|------|
| Decimal | `42`, `1_000_000` | `Int` |
| Hexadecimal | `0xFF`, `0x1A_2B` | `Int` |
| Binary | `0b1010_0101` | `Int` |
| Octal | `0o777` | `Int` |
| Float | `3.14`, `1_000.5` | `Float` |
| Scientific | `6.022e23`, `1.5e-10` | `Float` |
| Boolean | `true`, `false` | `Bool` |
| String | `"hello"`, `"value: {x}"` | `String` |

**Rationale:**
- **One Canonical Form**: Each base has exactly one prefix (`0x`, `0b`, `0o`). No suffixes — the type is determined by context or explicit annotation.
- **Explicit but Concise**: `_` separators improve readability (`1_000_000`). Scientific notation avoids hardcoding long float constants.
- **AI generation friendliness**: These literal formats are universal across Rust, Kotlin, Swift, Python — AI models are already trained on them. No novel syntax to hallucinate about.
- **No type suffixes**: Suffixes like `42u8` add ambiguity and complexity. Instead, use explicit type annotations when needed: `let b: UInt8 = 255`.

---

### 3. Character Type: No `Char`, `String` Only

**Options Considered:**
1. **`Char` exists** — `'a'` literal, represents a Unicode code point. Separate type from `String`.
2. **No `Char`** — All text is `String`. Single characters are just length-1 strings.

**Decision: Option 2 (no `Char`).**

**Rationale:**
- **One Canonical Form**: Only `"` for text. No `'a'` vs `"a"` confusion. AI never has to choose between single and double quotes.
- **Elixir lineage**: Elixir has no separate char type — strings and charlists serve different purposes, but oi simplifies this further by having only `String`.
- **Practical sufficiency**: Character-level processing (iteration, code points) is handled by `String` methods and `UInt8` for raw bytes. A dedicated `Char` type adds minimal value.
- **AI friendliness**: Eliminates an entire class of quoting errors in generated code.

---

### 4. Collection Literals: Tuple `()` Literal + Map/Set Constructors

**Options Considered:**
1. **Dedicated literals for all** — `#{}` for Map, `#[]` for Set, `()` for Tuple.
2. **Constructor-only** — `Map.of(...)`, `Set.of(...)`, `Tuple(...)`.
3. **Hybrid** — Tuple gets `()` literals; Map and Set use named constructors.

**Decision: Option 3 (hybrid).**

```oi
-- List literal (already established)
let items = [1, 2, 3]                              -- List[Int]

-- Tuple literal
let pair = (1, "hello")                             -- Tuple[Int, String]
let triple = (true, 42, "ok")                       -- Tuple[Bool, Int, String]

-- Map via constructor
let ages = Map.from([("Alice", 30), ("Bob", 25)])   -- Map[String, Int]

-- Set via constructor
let tags = Set.from(["urgent", "bug"])               -- Set[String]
```

**Rationale:**
- **One Canonical Form**: `()` for tuples is the only special literal. Map and Set have a single canonical constructor each — no competing syntax.
- **Tuple already in use**: Destructuring in `for (key, value) in entries` already implies tuple syntax. Making `()` the tuple literal is consistent.
- **Avoids symbol overload**: Dedicated Map/Set literals (`#{}`, `#[]`) add new sigils that compete with existing `{}` (blocks) and `[]` (generics, indexing). Named constructors are unambiguous.
- **AI friendliness**: `Map.from(...)` is self-documenting. An AI never needs to remember whether `#{}` is a Map or a Set.

---

### 5. Unit Type: `Unit` Keyword

**Options Considered:**
1. **`Unit` keyword** — Both type and value are `Unit`.
2. **`()` (empty tuple)** — Type = `()`, value = `()`.
3. **Hybrid** — Type = `Unit`, value = `()`.

**Decision: Option 1 (`Unit` keyword).**

```oi
-- Return type
fn log(msg: String) -> Unit
  with io
{
  io.println(msg)
}

-- if without else evaluates to Unit
if should_log {
  log.info("event")
}

-- In generics
fn save(user: User) -> Result[Unit, DbError]
  with db
{
  db.insert(user)
}
```

**Rationale:**
- **One Canonical Form**: `Unit` is both the type and the value. No ambiguity.
- **Consistency with naming convention**: oi types are PascalCase (`Int`, `String`, `Bool`, `Float`). `Unit` follows this pattern; `()` does not.
- **Avoids Tuple confusion**: Since `()` is now the tuple literal syntax, using `()` for Unit would create ambiguity — is `()` an empty tuple or Unit? Keeping them separate eliminates this.
- **Explicit**: `-> Unit` reads clearly as "this function returns nothing meaningful." `-> ()` requires knowledge of the convention.

## Consequences

### Positive
- The type system has a clear, small set of defaults (`Int`, `Float`, `Bool`, `String`, `Unit`) that cover the vast majority of code.
- Fixed-width types provide escape hatches for performance-critical and FFI scenarios without cluttering the default experience.
- No `Char` type eliminates an entire category of quoting ambiguity.
- `Unit` as a keyword is consistent with oi's PascalCase type naming.
- Collection creation follows a clear pattern: `[]` for lists, `()` for tuples, `Type.from(...)` for Map/Set.

### Trade-offs
- No arbitrary-precision integers by default. If needed, a `BigInt` library type can be added later.
- No `Char` means character-level operations require `String` methods, which may be slightly less efficient but is more consistent.
- Map/Set creation is slightly more verbose than languages with dedicated literals, but gains clarity.

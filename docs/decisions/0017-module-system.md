# ADR-0017: Module System

- Date: 2026-04-20
- Status: Accepted
- Participants: cbsuh, Antigravity (Claude Opus 4.6)

## Context
Before implementing the Phase 2 parser, oi needs definitive rules for how modules relate to files, how imports work, and how multi-file projects are structured. The existing `syntax-reference.md` had basic `module`/`use` examples but left three key questions unanswered: (1) module-file mapping, (2) `use` details (wildcards, aliasing), and (3) package/directory conventions.

## Decisions

### 1. One File = One Module (Explicit Declaration)

**Options Considered:**
1. **Implicit 1:1** — File name determines module name; `module` declaration optional.
2. **Explicit 1:1** — `module` declaration required on the first line; must match file name.
3. **Nested modules** — Multiple `module` blocks allowed within a single file.

**Decision: Option 2 (explicit 1:1).**

Rules:
- Every `.oi` file must begin with a `module` declaration.
- Exactly one `module` per file.
- File names use `snake_case`; module names use `PascalCase`. The compiler verifies consistency (`user_service.oi` → `module UserService`).
- No nested modules. Sub-modules are expressed through directory structure.

**Rationale:**
- **One Canonical Form**: There is exactly one way to define a module — the `module` keyword. No ambiguity about whether the module name comes from the file name or the declaration.
- **Explicit but Concise**: One line of overhead (`module Name`) provides full clarity. AI and human readers know the module's identity from the first line without checking the file name.
- **AI friendliness**: AI can generate complete, self-contained module files. No need to infer module names from file paths.
- **No nested modules**: Prevents the same logical grouping from being expressed two ways (nested `module` vs. separate files). Aligns with the flat-first project principle in `.cursorrules`.

---

### 2. Structured Imports + Aliasing, No Wildcards

**Options Considered:**
1. **Structured only** — Individual and grouped `{}` imports. No aliasing, no wildcards.
2. **Structured + aliasing** — Add `as` keyword for renaming imports.
3. **Structured + aliasing + wildcards** — Add `*` to import all public items.

**Decision: Option 2 (structured + aliasing, no wildcards).**

Supported import forms:
```oi
use core.Result                       -- Individual import
use core.{Result, Error}              -- Grouped import
use db.Connection as Conn             -- Aliased import
use auth.service.AuthService as Auth  -- Aliased deep import
```

**Not supported:**
```oi
-- use core.*                         -- ❌ Wildcards are forbidden
```

**Rationale:**
- **One Canonical Form**: Every imported symbol is explicitly listed. No `*` that hides what's actually used.
- **Explicit but Concise**: `as` enables practical brevity for name collisions and long type names without introducing ambiguity. It's a local name tag, not an alternative way to import.
- **AI friendliness**: AI can scan the `use` block and know every external dependency. With `*`, it would need to resolve the wildcard to know what symbols are available. Aliasing resolves name conflicts cleanly without requiring full-path usage throughout the code.
- **Human review friendliness**: Reviewers see every dependency at the top of the file. No hidden imports via `*`.

---

### 3. Directory = Namespace + Manifest for External Dependencies

**Options Considered:**
1. **Directory only** — Folder structure determines module paths. No manifest.
2. **Manifest only** — A config file maps module names to locations.
3. **Hybrid** — Directories for local modules; manifest (`oi.toml`) for external dependencies.

**Decision: Option 3 (hybrid).**

Local module resolution:
```
src/
  main.oi                   → module Main
  user_service.oi           → module UserService
  db/
    connection.oi           → module db.Connection
    query.oi                → module db.Query
  auth/
    service.oi              → module auth.Service
```

- The `src/` directory is the module root. Subdirectories create dotted namespaces.
- `use db.Connection` resolves to `src/db/connection.oi`.
- The directory name maps directly to the module path prefix (lowercase).

External dependencies (Phase 3+):
- An `oi.toml` file at the project root will declare external package dependencies.
- Detailed manifest syntax is deferred to Phase 3 (Package Manager / Build System).

**Rationale:**
- **One Canonical Form**: Module path = directory path. No configuration needed for local modules.
- **Explicit but Concise**: The directory structure is self-documenting. External dependencies require explicit declaration in `oi.toml`.
- **Practical**: Local development works without any config files. External dependency management is a separate concern addressed later.

## Consequences

### Positive
- Every `.oi` file is self-describing — the first line declares what it is.
- All imports are visible and explicit — no hidden dependencies from wildcards.
- The `as` keyword handles name conflicts and long paths gracefully.
- Directory structure maps 1:1 to module paths — no surprises.

### Trade-offs
- No nested modules means small utility types must go in their own files, which may feel like overhead for trivial helpers. However, this prevents "god files" with multiple unrelated modules.
- No wildcard `*` means more verbose `use` blocks for modules that import many items from one source. This is intentional — visibility is worth the extra lines.
- `oi.toml` details are deferred. External dependency management is not yet specified.

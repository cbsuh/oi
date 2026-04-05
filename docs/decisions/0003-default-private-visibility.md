# ADR-0003: Default Private Visibility with `pub`

- Date: 2026-04-05
- Status: Accepted
- Participants: cbsuh, AI (Claude Opus 4.6), AI (Gemini 3.1 Pro — identified the gap)

## Context
The module system defined `module` and `use` keywords but had no specification for access control. It was unclear whether functions and types were public or private by default, which would cause AI models to generate code with unpredictable visibility.

## Options Considered
1. **Default private + `pub` keyword** — Rust style. Everything is private unless explicitly marked `pub`.
2. **Default public + `internal` keyword** — Go/Python style. Everything is public unless explicitly restricted.
3. **Module-level `exports` list** — Haskell/OCaml style. The module header explicitly enumerates public items.

## Decision
**Option 1: Default private with `pub` keyword.**

## Rationale
- **Explicit but Concise**: Making something public is a deliberate, visible act. A single `pub` keyword is concise — no extra blocks or lists to maintain.
- **Effect Transparency**: The `pub` marker in the signature immediately tells AI and humans whether an item is part of the external API.
- **AI safety**: If AI forgets to add `pub`, the result is a private function (safe, restrictive). If the default were public, forgetting `internal` would accidentally expose internals (unsafe, permissive). Failing closed is safer.
- **One Canonical Form**: Only one keyword (`pub`) to learn. Option 2 requires `internal` and option 3 requires maintaining a separate exports list.

## Consequences
- All functions, types, and constants are private by default.
- `pub` is placed before `fn`, `type`, or `let` to make them externally accessible.
- AI code generators should only add `pub` when the function is explicitly intended as a public API.
- Module-internal helper functions need no annotation — they are private by nature.

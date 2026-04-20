# Target Market Notes

- Date: 2026-04-20
- Status: Discussion (leaning toward Game Data Cooking Pipeline as first target)

This document records the discussion around oi's first target market. No final decision has been made, but a leading candidate has emerged.

## Oi's Competitive Advantages

Structural advantages oi holds over existing languages:

| Strength | Core Value |
|----------|-----------|
| AI ↔ Human bidirectional collaboration | Write specs, AI implements, compiler verifies contracts |
| Effect Transparency (`with`) | Side effects declared in function signatures. Pure functions guaranteed |
| Design-by-Contract (`requires`/`ensures`) | Invariants enforced at the code level |
| One Canonical Form | Maximizes AI generation consistency, minimizes code review burden |
| Actor + Structured Concurrency | BEAM architecture + static types + effect tracking |
| Per-process GC | No global locks; independent GC per process |

## Candidate Markets

### 1. AI-Assisted Backend / Internal Tooling

- The only domain where **all five** of oi's core principles directly create value.
- Provides a language-level answer to "how do you trust AI-generated code?"
- Internal tooling carries low risk from language immaturity.
- Can start from Phase 2.

### 2. Game Server Logic

- Actor model maps naturally to game sessions, rooms, and players.
- Elixir has already validated this architecture at Riot Games (LoL), Discord, and others. Oi reinforces it with static types + effects + contracts.
- Contracts structurally enforce game economy invariants (e.g., preventing item duplication).
- Effect System separates pure game logic from IO → enables deterministic replay.
- Per-process GC is advantageous on servers over global GC (no global locks; heap freed wholesale on process termination).
- Prototyping possible from Phase 2 (BEAM itself is an interpreter).

### 3. Game Client Scripting (Engine Bindings)

- Actor ↔ game objects, Effects ↔ subsystems, Contracts ↔ game invariants — conceptually elegant mapping.
- Per-process GC structurally avoids Unity's notorious global GC spike problem (corrected from earlier assessment).
- However, making every game object an Actor is impractical due to message-passing overhead — only high-level entities (players, sessions) should be Actors; low-level objects (bullets, particles) should use plain data structures.
- Godot GDExtension is the most realistic entry path (open-source, officially supports language bindings).
- Ecosystem gaps (math/vector libraries) and strong incumbents (GDScript, C#, Lua) are challenges.
- Practical only after Phase 3–4.

### 4. AI Agent Orchestration

- Effect System for controlling agent behavior; Contracts for safety guarantees.
- Market is growing rapidly, but Python ecosystem is dominant — FFI/interop is a prerequisite.

### 5. Finance / Fintech

- Contracts shine brightest in this domain, but the market is very conservative.
- Suitable as a long-term target after Phase 4.

### 6. Packet Serialization (Cross-Cutting Capability)

Not a standalone market, but a **cross-cutting capability** that strengthens both the backend and game server markets.

**Core idea**: oi's type definitions serve as serialization schemas directly — no separate IDL (like `.proto` files) needed. `requires`/`ensures` provide data validation that protobuf and FlatBuffers lack. Structured metadata (`---` blocks) attach machine-parseable context to schemas.

**Protobuf vs FlatBuffers assessment**:

| Aspect | Protobuf | FlatBuffers |
|--------|----------|-------------|
| Deserialization | Copy-based (allocates new objects) | Zero-copy (reads from buffer directly) |
| Wire size | Small (varint encoding) | Large (alignment padding) |
| Oi GC compatibility | Good (GC manages decoded objects naturally) | Problematic (zero-copy requires pinned memory layout, conflicts with GC-managed heap per ADR-0012) |
| Contract integration | Natural (validate on decoded object) | Awkward (no object to validate against) |
| Ecosystem | Dominant (gRPC built on top) | Smaller (game-focused) |
| Implementation complexity | Straightforward | High (memory layout control needed) |

**Impact of encryption layers**: Modern systems increasingly add packet encryption (AES-GCM, etc.). This significantly changes the comparison:
- Decryption almost always allocates a new buffer, negating FlatBuffers' zero-copy advantage.
- Protobuf's smaller wire size becomes a double advantage: less data to transmit AND less data to encrypt/decrypt.
- FlatBuffers' larger payloads incur proportionally higher encryption cost.

**Conclusion**: Protobuf support first. FlatBuffers deferred until game client market is pursued and GC/memory-layout tensions are resolved.

**Possible approach**: oi struct/enum definitions with `@field` annotations auto-generate both internal serialization and `.proto` files for cross-language interop.

### 7. Game Data Cooking Pipeline ⭐ (Leading First-Target Candidate)

**The problem**: Game designers author data in human-friendly formats (Excel, CSV, JSON). This data must be loaded at game startup (client or server), but the designer-friendly format is inefficient for runtime use. The game parses, validates, and restructures data at startup — a process that dominates launch time. Current solutions are fragmented ad-hoc Python/C++ scripts with no systematic validation.

**The proposal**: oi as an offline build-time tool that reads designer data, validates it with contracts, transforms it with pure functions, and outputs a memory-mapped binary that C++ (or other runtimes) can load with zero parsing.

**Pipeline**:
```
Designer data (Excel/CSV/JSON)
  → oi pipeline (offline, build time)
    → Schema definition (oi structs with contracts)
    → Validation (requires/ensures)
    → Transformation (pure functions, AI-generatable)
    → Binary output (memory-mappable)
  → C++/game engine loads via mmap() — zero parsing, zero allocation
```

**Why every oi strength applies**:
- `requires`/`ensures`: Validate designer data at build time (e.g., drop probabilities sum ≤ 1.0, HP > 0). Catches errors before they reach runtime.
- Effect System: Separates pure transformation logic (no `with`) from IO (file reads/writes). Pure transforms are testable, parallelizable, and safe for AI generation.
- Structured metadata: Attach purpose, data source, ownership, and balance notes to schemas.
- AI collaboration: Designers/programmers write validation contracts, AI generates transformation logic.
- `concurrent.all`: Parallelize transformation of thousands of data entries.

**Why oi's weaknesses disappear**:
- GC performance: Irrelevant — offline tool, not real-time.
- Phase dependency: Phase 2 interpreter is sufficient for a build tool.
- Ecosystem: Minimal stdlib needed (file IO, CSV/JSON parsing, binary output).
- Competition: Weak — most studios use fragile internal scripts, not a mature product.

**Why this is a strong first target**:

| Aspect | AI-Assisted Backend (prev. #1) | Data Cooking Pipeline |
|--------|-------------------------------|----------------------|
| Stdlib needed | Heavy (HTTP, DB, JSON, env) | Minimal (file IO, CSV, binary) |
| Oi weaknesses exposed | Some (server perf, ecosystem) | None |
| Competition | Go, Python, TS (strong) | Ad-hoc scripts (weak) |
| Demo concreteness | Abstract ("trust AI code") | Concrete ("validate Excel → binary → load 0ms") |
| Gateway to next market | General backend | Game server → game client (natural expansion) |

**Output format**: Conceptually similar to FlatBuffers (pre-laid-out binary for direct memory access), but the GC conflict from section 6 does not apply here because oi runs offline — the output binary is consumed by C++, not by oi's runtime. Oi could emit FlatBuffers format, or define its own binary layout.

**Expansion path**: Team adopts oi for data pipeline → same team writes game server logic in oi → natural language adoption within the organization.

## Key Findings (Corrections Made During Discussion)

1. **Bytecode VMs are sufficient for game scripting** — Lua (WoW, Roblox), GDScript (Godot), and Squirrel (Valve titles) prove this. The real consideration is not the VM itself but oi's runtime characteristics.
2. **Per-process GC may not be a weakness for games** — Unity's global GC is notoriously problematic for game developers. Per-process GC has a smaller, more predictable lock scope, and short-lived processes skip GC entirely (heap freed on termination).
3. **Encryption layers negate FlatBuffers' zero-copy advantage** — Decryption produces a new buffer regardless, making the zero-copy path moot. Protobuf's smaller wire size becomes more valuable when encryption cost scales with payload size.
4. **Offline tooling eliminates oi's runtime concerns** — When oi runs as a build-time tool (not a runtime component), GC performance, Phase dependency, and ecosystem gaps all become irrelevant. The output is consumed by C++, not by oi.

## Open Questions

- Final first-target decision to be confirmed as Phase 2 implementation progresses.
- Binary output format for data cooking: custom layout, FlatBuffers-compatible, or both?
- Whether to prioritize game servers vs. general-purpose backend as the *second* target.
- How FFI/interop strategy influences market selection.
- Whether oi should define its own wire format or generate protobuf-compatible output for packet serialization.
- Serialization annotation design: `@field(n)` syntax, backward compatibility rules, deprecation markers.
- Minimum stdlib needed for data cooking pipeline: CSV parser, JSON parser, binary writer, file IO.

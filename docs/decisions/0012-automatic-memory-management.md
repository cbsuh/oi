# ADR-0012: Automatic Memory Management (GC-Based)

- Date: 2026-04-20
- Status: Accepted
- Participants: cbsuh, Claude Opus 4.6 (Thinking)

## Context

As oi moves toward implementation (Phase 2+), the memory management strategy must be decided. This is a foundational architectural choice that affects runtime design, performance characteristics, and — most importantly — whether the language syntax needs to expose any memory-related constructs.

Three primary approaches exist in modern languages:
1. Manual memory management (C, Zig) — `malloc`/`free`
2. Ownership-based automatic management (Rust) — borrow checker, lifetimes
3. Garbage collection (Go, Java, Erlang/Elixir) — runtime-managed

## Options Considered

1. **Manual malloc/free** — The programmer explicitly allocates and frees memory.
2. **Ownership + borrow checker (Rust-style)** — Compile-time memory safety via lifetimes, without GC.
3. **Automatic GC** — The runtime handles all memory allocation and deallocation transparently.

## Decision

**Option 3: Automatic GC.** Memory management is entirely handled by the runtime and is invisible to the programmer. No memory-related syntax is exposed in the language.

Long-term target: **per-process GC** (BEAM/Erlang-style), where each actor/process has its own isolated heap and performs independent garbage collection. See ADR-0013 for the concurrency model that motivates this design.

### Phased Implementation

| Phase | Memory Strategy |
|-------|----------------|
| Phase 2 (Tree-Walking Interpreter) | Host language (Rust) manages memory |
| Phase 3 (Bytecode VM) | Single-heap tracing GC as starting point |
| Phase 4 (Native Backend) | Per-process GC aligned with actor model |

## Rationale

### Against manual malloc/free (Option 1)

- **AI generation risk**: An AI forgetting `free` produces memory leaks; forgetting to null a pointer produces use-after-free. These are the most dangerous classes of bugs.
- **Contradicts oi's principles**: oi's "Explicit but Concise" principle rejects unnecessary boilerplate. Memory management syntax adds cognitive overhead without expressing business intent.
- **Already rejected**: `comparison.md` explicitly states oi "sacrifices micro-optimization features" for simplicity.

### Against ownership/borrow checker (Option 2)

- **Cognitive overhead**: Rust's borrow checker is powerful but introduces significant learning curve and "fighting the borrow checker" friction.
- **Already rejected**: `comparison.md` states oi "replaces Rust's borrow checker mental overhead with higher-level semantic safety checks."
- **AI unfriendly**: Lifetime annotations are a frequent source of AI hallucination in Rust code generation.

### For automatic GC (Option 3)

- **One Canonical Form**: No memory-related syntax means zero ambiguity — there is no "wrong way" to manage memory.
- **Explicit but Concise**: Business logic is not polluted with allocation concerns.
- **AI generation friendly**: AI never needs to reason about memory; it focuses purely on logic and effects.
- **Human review friendly**: Reviewers never need to audit memory safety.
- **Immutable-first design synergy**: oi's `let`-by-default (ADR-0002) means no circular references in typical code, making GC straightforward and efficient.

### Why per-process GC as the long-term target

- Each actor/process owns an isolated heap (see ADR-0013).
- GC pauses affect only the individual process, not the entire application.
- Short-lived processes can skip GC entirely — the heap is freed wholesale on termination.
- No shared memory means no locks, no concurrent GC complexity.
- Proven at scale by BEAM VM (Erlang/Elixir) over 30+ years.

## Consequences

### Positive
- **Zero memory syntax**: The language surface remains clean. No `malloc`, `free`, `Box`, `Rc`, `Arc`, lifetime annotations, or `unsafe` blocks.
- **Safe by default**: Memory leaks and use-after-free are structurally impossible at the language level.
- **Implementation flexibility**: The GC strategy can evolve across phases without any language syntax changes, since memory management is never exposed to the user.

### Trade-offs
- **Less control**: Programmers cannot optimize memory layout for cache-friendly access patterns. This is an intentional trade-off — oi targets correctness and clarity over micro-optimization.
- **GC pauses**: Even with per-process GC, long-lived processes with large heaps may experience pauses. Mitigation strategies (hibernation, process recycling) will be documented as patterns.
- **Large shared data**: Pure per-process isolation requires data copying between actors. For large shared datasets, an auxiliary mechanism (similar to Erlang's ETS) may be needed in the future.
- **Runtime complexity**: Implementing per-process GC requires a sophisticated runtime scheduler, which increases implementation effort in Phase 3-4.

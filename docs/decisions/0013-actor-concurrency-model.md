# ADR-0013: Actor-Based Concurrency Model

- Date: 2026-04-20
- Status: Accepted
- Participants: cbsuh, Claude Opus 4.6 (Thinking)

## Context

oi needs a concurrency model that aligns with its core principles: One Canonical Form, Explicit but Concise, and Effect Transparency. The model must also be compatible with the per-process GC strategy chosen in ADR-0012.

Additionally, the concurrency model should avoid the "colored function" problem (async/await), minimize shared-state bugs, and be natural for AI to generate and humans to review.

## Options Considered

1. **Actor Model** (Erlang/Elixir) — Isolated processes communicating via message passing.
2. **CSP (Communicating Sequential Processes)** (Go) — Goroutines with shared channels.
3. **async/await** (Rust, JS, Kotlin) — Cooperative coroutines with explicit suspension points.
4. **Actor + Structured Concurrency hybrid** — Actors for stateful entities, structured concurrency for stateless parallelism.

## Decision

**Option 4: Actor + Structured Concurrency hybrid.**

- **`actor`**: For stateful, long-lived concurrent entities. Each actor owns its state and communicates exclusively via typed messages.
- **`concurrent`**: For stateless parallel execution of independent tasks, with structured scoping that guarantees completion.

Both mechanisms are integrated into oi's effect system via `with spawn` and `with concurrent`.

### Conceptual Syntax (subject to refinement)

#### Actor — stateful concurrent entity

```oi
actor CounterActor {
  state: Int = 0

  on Increment { amount: Int } {
    state = state + amount
  }

  on Decrement { amount: Int } {
    state = state - amount
  }

  on GetCount -> Int {
    state
  }
}

fn main() with spawn, io {
  let counter = spawn(CounterActor)
  counter.send(Increment { amount: 5 })
  counter.send(Increment { amount: 3 })
  let count = counter.ask(GetCount)?
  io.println("Count: {count}")
}
```

#### Structured Concurrency — stateless parallelism

```oi
fn fetch_dashboard(user_id: UserId) -> Dashboard
  with concurrent, db, network
{
  let (profile, orders, notifications) = concurrent.all(
    || db.get_profile(user_id)?,
    || db.get_orders(user_id)?,
    || network.get_notifications(user_id)?
  )
  Dashboard { profile, orders, notifications }
}
```

### Role Separation

| Use Case | Mechanism | Effect | Example |
|----------|-----------|--------|---------|
| Stateful long-lived entity | `actor` + `spawn` | `with spawn` | Cache, session manager, rate limiter |
| Stateless parallel tasks | `concurrent.all` | `with concurrent` | Parallel API calls, batch processing |
| Stream/pipeline processing | `\|>` + actor | existing syntax | Data transformation pipelines |

## Rationale

### Against CSP / Go-style channels (Option 2)

- **Shared channels violate isolation**: Channels are shared mutable state, conflicting with oi's immutable-first design and per-process GC (ADR-0012).
- **Deadlock-prone**: AI-generated code with channels can easily create deadlocks via circular channel dependencies.
- **No canonical form**: Multiple goroutines reading from the same channel creates race-like semantics that are hard to review.

### Against async/await (Option 3)

- **Colored function problem**: Once a function is `async`, all callers must also be `async`. This creates a viral annotation burden that contradicts "Explicit but Concise."
- **No state encapsulation**: async/await provides concurrency but no model for shared state management.
- **Incompatible with per-process GC**: async/await assumes a shared heap with cooperative scheduling, not isolated processes.

### Against pure Actor only (Option 1)

- **Overhead for simple parallelism**: When you just want to run 3 independent tasks in parallel, defining an actor with message types is excessive.
- **Request-response verbosity**: Simple "call and get result" patterns require message type definitions, send, and receive boilerplate.

### For Actor + Structured Concurrency hybrid (Option 4)

- **One Canonical Form**: Clear decision rule — has state? → `actor`. No state, just parallel? → `concurrent`. No ambiguity.
- **Effect Transparency**: `with spawn` and `with concurrent` make concurrency intentions visible in function signatures. A function without these effects is guaranteed sequential.
- **Per-process GC alignment**: Each actor maps to a process with its own heap (ADR-0012). `concurrent` tasks can also map to short-lived processes that are immediately reclaimed.
- **AI generation friendly**: Both patterns are highly structured and template-like. AI can reliably generate actor definitions and concurrent blocks.
- **Human review friendly**: Reviewers can understand actor behavior by reading its message handlers sequentially. `concurrent.all` makes parallelism explicit and bounded.
- **No colored functions**: Neither `actor` nor `concurrent` requires annotating the calling function as "async." The effect system (`with`) handles this cleanly.

## Consequences

### Positive
- **Concurrency is always explicit**: No hidden threads or implicit parallelism. The effect system tracks all concurrent behavior.
- **State isolation**: Mutable state exists only inside actors, extending oi's "local-only mutation" rule (ADR-0002) to the concurrent domain. Module-level `var` remains forbidden; actors are the sanctioned alternative.
- **Structured completion**: `concurrent.all` guarantees all tasks complete (or fail) before the scope exits. No orphaned tasks.
- **Natural GC synergy**: Actor lifecycle directly maps to per-process heap lifecycle (ADR-0012).

### Trade-offs
- **Two concurrency primitives**: Having both `actor` and `concurrent` is more complex than a single unified model. However, each serves a distinct, non-overlapping purpose with a clear selection criterion.
- **Actor deadlock risk**: Circular `ask` calls between actors can deadlock. Mitigation: documentation of patterns, and potentially a compile-time cycle detector in the future.
- **Message type boilerplate**: Each actor requires explicit message type definitions. Mitigation: the `on` syntax inlines message definitions, reducing separate type declarations.
- **Deferred syntax**: The exact syntax for `actor`, `on`, `spawn`, and `concurrent` is conceptual and will be refined during Phase 2 implementation. The architectural decision (actor + structured concurrency) is stable; the syntax is not yet final.

### Future Considerations
- **Supervision trees**: Erlang-style supervisor patterns for fault tolerance may be introduced later.
- **Shared data store**: An ETS-like mechanism for large shared datasets may be needed (see ADR-0012 trade-offs).
- **Actor discovery**: Naming and registry for actors (similar to Erlang's registered processes) is deferred.
- **Distributed actors**: Cross-node actor communication is out of scope for now but the model does not preclude it.

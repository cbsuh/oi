# Design Principles

The `oi` language is purpose-built for an era where AI generates code and humans review, maintain, and architect it. To achieve this, it adheres strictly to five core principles.

## Philosophical Lineage

`oi` is not designed in a vacuum. Its architectural foundation — actor-based concurrency, per-process garbage collection, immutability by default, and pipeline-oriented data flow — comes directly from **Elixir/Erlang and the BEAM ecosystem**, a platform proven at scale over 30+ years.

Where `oi` diverges is not in philosophy but in **reinforcement**. Elixir's core ideas are sound, but its dynamic type system, invisible side effects, and inconsistent error handling create friction in practice — especially for AI-generated code. `oi` addresses these gaps with a static type system, an explicit effect system (`with`), unified error handling (`Result[T, E]`), and Design-by-Contract (`requires`/`ensures`).

In short: **Elixir got the architecture right. `oi` adds the guardrails.**

## 1. One Canonical Form

There should be exactly one correct way to write any given block of logic.
- **Why**: AI models benefit from clear, unambiguous target formats. When humans read the code, predictable structures massively reduce cognitive load.
- **How**: Formatting (like `gofmt` or `rustfmt`) is built into the language definition. Redundant keywords or aliases are rejected by design.
- **For AI**: No need to worry about formatting choices during code generation.
- **For Humans**: Eliminates style debates; consistent code improves teamwork.
- **Example**: Avoids ambiguities like C++'s `>>` which could mean either bitshift or template closing depending on context.

## 2. Explicit but Concise

Implicit behaviors—like hidden type conversions or dynamic scoping—are frequent sources of bugs and AI hallucinations.
- **Why**: The code must show exactly what is executing. At the same time, boilerplate reduces readability.
- **How**: We avoid implicit behavior but use powerful language constructs (like pattern matching and structural typing) to keep the code footprint small. 
- **For AI**: Types, effects, and errors are all visible in signatures, making reasoning straightforward.
- **For Humans**: Focus on core business logic without wading through unnecessary boilerplate.
- **Example**: Prevents Python's implicit `__init__` calls and JavaScript's implicit type coercion (`"2" + 2 = "22"`).

## 3. Intent-Declarative

Code usually explains *how* it does something, but `oi` requires you to explain *what* and *why*.
- **Why**: An AI needs rigorous boundaries to generate correct implementations, and humans need to quickly verify the intent of AI-generated code.
- **How**: First-class support for Contracts. `requires` (preconditions) and `ensures` (postconditions) are seamlessly integrated.
- **For AI**: Contracts provide precise boundaries for automatic code generation and self-testing.
- **For Humans**: Understand a function's purpose from its specification alone, without reading the implementation.

```oi
fn divide(a: Float, b: Float) -> Float
  requires b != 0.0
  ensures result => result * b == a
{
  a / b
}
```

## 4. Structured Metadata

Documentation should be parseable by both humans and machines.
- **Why**: Free-form comments are often ignored by automated tools or misinterpreted by LLMs.
- **How**: We use structured attributes and machine-parseable YAML-style annotations (`---` blocks) to provide context.
- **For AI**: Fields like `ai_notes` automatically convey cautions and constraints when modifying code.
- **For Humans**: The `purpose` field instantly communicates why a function exists.

```oi
---
purpose: "이율을 바탕으로 최종 가격을 계산한다"
ai_notes: "이 로직을 수정할 때 기존 rate 정책이 깨지지 않게 유의"
---
fn calculate_price(...) { ... }
```

## 5. Effect Transparency

Side effects must be strictly tracked and visible.
- **Why**: It is difficult to review code without knowing if a function modifies state, reads a file, or makes a network call.
- **How**: All side-effects are declared at the signature level using the `with` keyword.
- **For AI**: Effect information enables safe refactoring and optimization without hidden breakage.
- **For Humans**: A single glance at the function signature reveals which parts of the system it touches.

```oi
fn fetch_data(url: String) -> String with network {
  network.http_get(url)
}
```

---

## Additional Goals

Beyond the five core principles, `oi` pursues two higher-level goals that emerge naturally from its design.

### Self-Verifiable Code

The `requires`/`ensures` contracts are not just documentation — they are a mechanism for the compiler to **automatically verify** that generated code meets its specification.

- **For AI**: This creates a feedback loop. An AI generates code → the compiler checks contracts → the AI knows whether its output is correct without waiting for a human. It turns open-ended code generation into a constrained, verifiable problem.
- **For Humans**: The reviewer's job shifts from "is this code correct?" to "are these contracts sufficient?" — a much smaller surface to audit.

```oi
fn binary_search[T: Comparable](list: List[T], target: T) -> Result[Int, NotFound]
  requires list.is_sorted()
  ensures result.is_ok() => list[result.value] == target
{
  -- AI generates the implementation
  -- Compiler verifies it satisfies the contract
}
```

### Bidirectional Collaboration

The core workflow is "AI generates, human reviews." But `oi` also supports the **reverse**: humans write the specification (signatures + contracts), and AI fills in the implementation.

- **For AI**: Instead of interpreting ambiguous natural language, the AI receives a precise, machine-readable specification. This dramatically reduces hallucination and narrows the solution space.
- **For Humans**: Writing a signature with `requires`/`ensures` is faster than writing the full implementation, and the contract serves as living documentation.

```oi
-- Human writes this:
fn sort[T: Comparable](list: List[T]) -> List[T]
  ensures result.length == list.length
  ensures result.is_sorted()

-- AI fills in:
{
  -- implementation generated by AI
}
```

This bidirectional flow makes `oi` a true **collaboration language**, not just a one-way generation target.

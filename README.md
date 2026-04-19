# oi

A programming language designed for AI-human collaboration.

> **oi** is currently in the **design phase**. The language specification is being
> developed and no compiler or interpreter exists yet.

## Design Philosophy

oi is built on five core principles optimized for the AI-generates, human-reviews workflow:

1. **One Canonical Form** — There is exactly one way to express any given meaning. Formatting is part of the language spec.
2. **Explicit but Concise** — No implicit behavior, but the language absorbs repetitive patterns so code stays short.
3. **Intent-Declarative** — Express *what* and *why*, not just *how*. Contracts (`requires`/`ensures`) are first-class.
4. **Structured Metadata** — Machine-parseable annotations instead of free-form comments.
5. **Effect Transparency** — Side effects are visible at the type level via the `with` keyword.

Beyond these principles, oi pursues two additional goals:

- **Self-Verifiable Code** — The compiler uses `requires`/`ensures` contracts to automatically verify generated code, giving AI a built-in feedback loop and reducing the human reviewer's burden.
- **Bidirectional Collaboration** — Humans write signatures and contracts; AI fills in the implementation. Both directions of the AI-human workflow are first-class.

## Quick Look

```
fn fizzbuzz(n: Int) -> String {
  match (n % 3, n % 5) {
    (0, 0) => "FizzBuzz"
    (0, _) => "Fizz"
    (_, 0) => "Buzz"
    _      => n.to_string()
  }
}

fn main()
  with io
{
  List.range(1, 101)
    |> map(fizzbuzz)
    |> each(|s| io.println(s))
}
```

## Current Status

| Phase | Status |
|-------|--------|
| Language Design | 🟡 In Progress |
| Parser / Lexer | ⬜ Not Started |
| Interpreter | ⬜ Not Started |
| Compiler | ⬜ Not Started |

## Documentation

- [Design Principles](docs/design-principles.md)
- [Syntax Reference](docs/syntax-reference.md)
- [Effect System](docs/effect-system.md)
- [Error Handling](docs/error-handling.md)
- [Comparison with Other Languages](docs/comparison.md)
- [Roadmap](docs/roadmap.md)
- [Design Decisions (ADRs)](docs/decisions/)

## Examples

See the [examples/](examples/) directory for sample oi programs.

## Contributing

We welcome contributions of all kinds! See [CONTRIBUTING.md](CONTRIBUTING.md) for details.

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
- MIT License ([LICENSE-MIT](LICENSE-MIT))

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.

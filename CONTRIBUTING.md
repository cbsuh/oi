# Contributing to oi

Thank you for your interest in contributing to oi! This project is in its early
design phase, and every kind of contribution is valuable.

## How You Can Help Right Now

Since oi is currently in the design phase (no compiler or interpreter yet),
here are the most impactful ways to contribute:

- **Language design feedback** — Read the docs in [`docs/`](docs/) and open
  GitHub Issues with your thoughts, questions, or suggestions.
- **Example programs** — Propose new examples in [`examples/`](examples/) to
  explore how the language feels in practice.
- **RFC proposals** — Write a formal proposal for a new language feature using
  the template in [`rfcs/0001-template.md`](rfcs/0001-template.md).
- **Documentation improvements** — Fix typos, clarify explanations, or improve
  formatting.
- **Join discussions** — Participate in GitHub Discussions to help shape the
  direction of the language.

## Making Changes

1. Fork the repository and create a branch from `main`.
2. Make your changes.
3. Ensure your changes follow the existing style and conventions.
4. Submit a Pull Request with a clear description of *what* you changed and *why*.

## RFC Process

To propose a new language feature:

1. Copy `rfcs/0001-template.md` to `rfcs/NNNN-my-feature.md` (pick the next
   available number).
2. Fill in the template with your proposal.
3. Submit a Pull Request. The PR will serve as the discussion thread.
4. After discussion, the proposal will be accepted, revised, or rejected.

## Design Decision Records (ADR)

All design decisions — whether made through discussion, RFC resolution, or
AI-assisted brainstorming — must be recorded as an ADR:

1. Copy `docs/decisions/0000-template.md` to `docs/decisions/NNNN-topic.md`.
2. Document the context, options considered, decision, rationale, and consequences.
3. Include a `Participants` field listing everyone (human or AI) involved.
4. Submit as part of the PR that implements the decision.

See existing ADRs in [`docs/decisions/`](docs/decisions/) for examples.

## License Agreement

By contributing to oi, you agree that your contributions will be licensed under
the project's dual license:

> Your contribution is licensed under both the **MIT License** and the
> **Apache License, Version 2.0**, at the user's option.
>
> You grant this license without any additional terms or conditions.

This is the same approach used by the Rust programming language.

## Code of Conduct

All participants in this project are expected to follow our
[Code of Conduct](CODE_OF_CONDUCT.md). Please report unacceptable behavior.

## Questions?

Feel free to open a GitHub Issue or start a Discussion if you have any questions.
We're happy to help!

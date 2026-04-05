---
description: Make a design decision for the oi language through structured evaluation
---

# Design Decision Workflow

When the user invokes this workflow with a topic (e.g., `/design-decision string interpolation syntax`), follow these steps exactly:

## 1. Check Existing ADRs
// turbo
- Read all files in `docs/decisions/` to understand past decisions.
- If any existing ADR is directly relevant to the topic, summarize it and ask whether this is a revision or a new decision.

## 2. Research Options
- Identify 2-4 concrete options for the decision.
- For each option, provide:
  - A short name and description
  - Which language(s) use this approach
  - A code example in oi syntax

## 3. Evaluate Against oi Principles
- Create a comparison table evaluating each option against:
  - One Canonical Form
  - Explicit but Concise
  - Intent-Declarative (if applicable)
  - Effect Transparency (if applicable)
  - AI generation friendliness
  - Human review friendliness

## 4. Recommend and Wait
- State your recommendation with a clear rationale.
- **STOP and wait for user approval.** Do not proceed until the user explicitly confirms.

## 5. Create ADR (after approval)
- Create `docs/decisions/NNNN-topic-name.md` using the template at `docs/decisions/0000-template.md`.
- Use the next available number. Check existing files to determine it.
- Include `Participants` with the user's name and the AI model name.

## 6. Update Documentation (after approval)
- Update these files if the decision affects them:
  - `docs/syntax-reference.md` — add/modify syntax sections
  - `docs/design-principles.md` — if a principle is affected
  - `docs/effect-system.md` — if effects are affected
  - `docs/error-handling.md` — if error handling is affected
  - `.cursorrules` — add/update AI code generation guidelines
  - `llms.txt` — add/update technical constraints
  - `examples/` — add example code if a new syntax feature is introduced
- After updating, list exactly which files were modified and what changed.

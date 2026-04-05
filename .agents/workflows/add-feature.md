---
description: Add a new syntax feature to the oi language with full documentation
---

# Add Feature Workflow

When the user invokes this workflow with a feature description (e.g., `/add-feature for loop syntax`), follow these steps exactly:

## 1. Check for Conflicts
// turbo
- Read all files in `docs/decisions/` for existing ADRs.
- Read `docs/syntax-reference.md` to check if the feature already exists.
- Read `docs/comparison.md` "What we Discarded" table — if the proposed feature was explicitly discarded, warn the user immediately and stop.
- If any conflict is found, report it and ask the user how to proceed before continuing.

## 2. Design Decision
- Follow the `/design-decision` workflow for this feature:
  - Present 2-4 options with code examples
  - Evaluate against oi principles
  - Recommend and wait for user approval
- If the user has already specified exactly what they want (no ambiguity), skip option evaluation and proceed to confirmation.

## 3. Create ADR (after approval)
- Create `docs/decisions/NNNN-feature-name.md` using the template.
- Use the next available ADR number.

## 4. Update syntax-reference.md
- Add a new section (or update an existing one) in `docs/syntax-reference.md`.
- Include:
  - Clear description of the feature
  - At least 2 code examples (one simple, one realistic)
  - Any restrictions or rules

## 5. Update AI Context Files
- `.cursorrules`: Add a numbered guideline under "Guidelines for Code Generation and Assistance" if the feature has rules AI must follow.
- `llms.txt`: Add a constraint line under "Technical constraints for generating `.oi` code" if applicable.

## 6. Update Other Documentation (if affected)
Check and update these files if the feature touches their domain:
- `docs/design-principles.md` — if it relates to core principles
- `docs/effect-system.md` — if it involves effects
- `docs/error-handling.md` — if it involves error types
- `docs/comparison.md` — if it changes comparison with other languages

## 7. Add Example Code
- Create or update an example file in `examples/` that demonstrates the new feature in a realistic context.

## 8. Verify (run /review-docs)
- After all changes, perform the `/review-docs` workflow to check for consistency issues introduced by the changes.
- Report the final verification results.

## Summary
After completion, list all files created and modified in a table:

| Action | File | What Changed |
|--------|------|-------------|
| Created | docs/decisions/NNNN-... | ADR for the decision |
| Modified | docs/syntax-reference.md | Added section for ... |
| ... | ... | ... |

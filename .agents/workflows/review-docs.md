---
description: Review all oi project documentation for consistency and AI-readiness
---

# Document Review Workflow

When the user invokes `/review-docs`, perform a comprehensive review of all project files.

## 1. Read All Project Files
// turbo
Read every file in the following list. Do not skip any:
- `README.md`
- `llms.txt`
- `.cursorrules`
- `CONTRIBUTING.md`
- `docs/design-principles.md`
- `docs/syntax-reference.md`
- `docs/effect-system.md`
- `docs/error-handling.md`
- `docs/comparison.md`
- `docs/roadmap.md`
- All files in `docs/decisions/`
- All files in `examples/`

## 2. Apply Checklist
For each file, check the following:

### Code Examples
- [ ] All code blocks use valid oi syntax (no missing operators, no wrong brackets)
- [ ] `ensures` clauses use `=>` (not a bare space)
- [ ] All `Result` types use square brackets: `Result[T, E]` (never `<T, E>`)
- [ ] Functions with side-effects have `with` declarations
- [ ] `var` is only used inside function bodies (never in parameters, module-level, or type fields)
- [ ] `pub` is used correctly (not applied to internal helpers)

### Language Consistency
- [ ] Documentation text is in a single language (English for docs/, Korean is OK in `purpose` metadata values)
- [ ] Technical terms are consistent across all files (same keywords, same capitalization)

### Completeness
- [ ] Every syntax feature in `syntax-reference.md` has at least one code example
- [ ] Every rule in `.cursorrules` is also reflected in `llms.txt`
- [ ] Every ADR decision is reflected in the relevant documentation files
- [ ] All `examples/*.oi` files use current syntax (match latest syntax-reference)

### Cross-File Consistency
- [ ] `README.md` design philosophy matches `design-principles.md`
- [ ] `.cursorrules` guidelines match `syntax-reference.md` rules
- [ ] `llms.txt` constraints match `.cursorrules` guidelines
- [ ] `comparison.md` "discarded features" list does not contradict any syntax feature

## 3. Report Results
Present findings sorted by severity:
- 🚨 **Critical**: Syntax errors in code examples, contradictions between files
- ⚠️ **High**: Missing rules in `.cursorrules`/`llms.txt`, undocumented features
- ℹ️ **Medium**: Missing examples, minor inconsistencies, stale references

For each finding, specify the exact file and line number.

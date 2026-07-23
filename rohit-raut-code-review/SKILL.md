---
name: rohit-raut-code-review
description: >
  Four-pass frontend code review: standards+spec, accessibility, over-engineering, then craft quality.
  Orchestrates code-review → analyze-component-accessibility → ponytail-review → uncle-bob-craft in sequence.
  Also checks test coverage obligations.
  Use when the user says "full review", "review my changes", "review since X", or invokes /rohit-raut-code-review.
---

# Full Review

Sequential four-pass review pipeline with housekeeping checks. Each pass has a gate — only proceed if the gate opens.

## Process

### 0. Housekeeping checks

Run these before the main passes. Conditional on what the repo supports.

**Test check:**
Detect if the repo has a test runner — look for a `test` script in `package.json`, or directories matching `__tests__/`, `*.test.*`, `*.spec.*`, `tests/`, `test/`. If tests are supported:
- Check whether the diff includes any new or modified test files.
- If the diff adds or modifies non-test source files but includes zero test changes, flag: "⚠️ No test additions/changes — consider adding coverage for new logic."

If no test infrastructure is detected, skip silently.

**Completion criterion:** All checks reported (flagged or skipped).

### 1. Standards & Spec review

Invoke the `code-review` skill (two-axis: Standards + Spec).

- If the user supplied a fixed point, pass it through.
- If not, ask for one before proceeding.

**Completion criterion:** Standards and Spec reports are both presented to the user.

### 2. Accessibility review (conditional)

**Gate:** the diff contains changes to rendering — files matching `*.jsx`, `*.tsx`, or template markup in `*.js`/`*.ts` that returns JSX/HTML. Pure logic changes (utilities, reducers, API calls, configs, tests) skip this pass entirely.

How to check the gate:
```
git diff <fixed-point>...HEAD --name-only | grep -E '\.(jsx|tsx)$'
```
If that returns nothing, also spot-check `.ts`/`.js` files in the diff for JSX return statements. If still nothing renders, skip and note "No rendering changes — a11y pass skipped."

When the gate opens, invoke the `analyze-component-accessibility` skill for each changed component that renders UI. Use focus=`general`, depth=`standard`, level=`AA`.

**Completion criterion:** Every changed rendering file has been checked, or the gate closed with a skip note.

### 3. Ponytail review (over-engineering)

Invoke the `ponytail-review` skill against the same diff.

**Completion criterion:** Ponytail findings list (or "Lean already. Ship.") is presented.

### 4. Craft quality (Uncle Bob)

Invoke the `uncle-bob-craft` skill against the same diff. Focus on:

1. **Dependency Rule & boundaries** — do dependencies point inward? Is business logic free from framework/UI/DB imports?
2. **SOLID in context** — any violations in the touched code? Name the principle and the file/function.
3. **Naming & readability** — are functions/variables intention-revealing? Is there opacity (hard to understand at a glance)?
4. **Function size & responsibility** — functions doing more than one thing, mixed levels of abstraction.
5. **Smells** — rigidity, fragility, needless complexity, needless repetition, opacity. Name the smell and location.
6. **Concrete suggestions** — for each finding, propose a specific refactor (extract function, rename, invert dependency, split responsibility).

**Gate:** Always runs. Every diff benefits from craft review.

**Completion criterion:** Craft findings list (or "Clean code. Ship.") is presented.

### 5. Summary

End with a combined tally:

```
## Review Complete

| Pass              | Findings |
|-------------------|----------|
| Housekeeping      | N warnings (or ✓) |
| Standards         | N        |
| Spec              | N        |
| Accessibility     | N (or skipped) |
| Over-engineering  | N (net: -X lines) |
| Craft (Uncle Bob) | N (or ✓) |
```

One sentence: the single worst finding across all passes, if any.

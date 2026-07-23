---
name: ponytail-review
description: >
  Code review focused exclusively on over-engineering. Finds what to delete:
  reinvented standard library, unneeded dependencies, speculative abstractions,
  dead flexibility. Use when the user says "review for over-engineering", "what
  can we delete", "is this over-engineered", "simplify review", or invokes
  /ponytail-review. Complements correctness-focused review, this one only
  hunts complexity.
---

Review diffs for unnecessary complexity. The diff's best outcome is getting shorter.

## Output Format

Present findings as a numbered list. Each item has:
- **Location** — file and line(s)
- **Issue** — what's wrong (tagged with category)
- **Fix** — what to do instead

### Categories

- `delete` — dead code, unused flexibility, speculative feature
- `stdlib` — hand-rolled thing the standard library already ships
- `native` — dependency or code doing what the platform already does
- `yagni` — abstraction with one implementation, config nobody sets, layer with one caller
- `shrink` — same logic possible in fewer lines

### Example Output

---

**1. [stdlib] `src/utils/email.ts` L12-38**
- Issue: 27-line EmailValidator class that just checks if "@" is present.
- Fix: Replace with `email.includes("@")` — 1 line. Real validation is the confirmation mail anyway.

**2. [native] `src/helpers/date.ts` L4**
- Issue: `moment.js` imported for a single format call.
- Fix: Use `Intl.DateTimeFormat` — 0 deps, built into the platform.

**3. [yagni] `repo.py` L88**
- Issue: `AbstractRepository` base class with only one implementation.
- Fix: Inline the concrete class directly. Add the abstraction back when a second implementation actually exists.

**4. [delete] `src/api/retry.ts` L52-71**
- Issue: Retry wrapper around an idempotent local call that can't fail transiently.
- Fix: Delete it. Nothing replaces it.

**5. [shrink] `src/transform.ts` L30-44**
- Issue: Manual loop that builds a dict from two arrays.
- Fix: `Object.fromEntries(keys.map((k, i) => [k, values[i]]))` — 1 line.

---

## Scoring

End every review with: `net: -<N> lines possible.`

If there is nothing to cut, say `Lean already. Ship.` and stop.

## Boundaries

Scope: over-engineering and complexity only. Correctness bugs, security holes,
and performance are explicitly out of scope — route them to a normal review.
A single smoke test or assert-based self-check is the ponytail minimum, not bloat — never flag it for deletion.
Does not apply the fixes, only lists them.

"stop ponytail-review" or "normal mode": revert to verbose review style.

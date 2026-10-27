---
name: slop-check
description: >
  Audits the current branch diff for poor quality and AI-generated "slop" — LLM output
  merged without meaningful human review. Runs git diff against the merge base, then
  produces a structured report with named smell codes, severity ratings, and an AI Slop
  Confidence Score. Use when user says "slop check", "check for slop", "audit this branch",
  "review for AI slop", or invokes /slop-check. Auto-triggers when asked to audit
  code quality or check for AI-generated patterns on the current branch.
---

You are a senior code reviewer. Audit the current branch diff for poor quality and AI-generated slop. No praise. No sandwich. Audit only.

## Step 1: Get the diff

Run these in order until you have a non-empty diff:

```bash
git diff $(git merge-base HEAD main)..HEAD
```

If `main` fails, try `master`. If still empty, try:

```bash
git diff HEAD~1..HEAD
```

If the diff is empty, tell the user there is nothing to audit and stop.

## Step 2: Audit

You are looking for two overlapping categories:

1. **General poor quality** — buggy, fragile, over-engineered, unmaintainable
2. **AI slop signals** — named smells below; reference by code in report

### Smell codes

**Structural**
- `OVER_ABSTRACTION` — unnecessary interfaces, base classes, factories for trivial logic
- `PREMATURE_GENERALITY` — generic params, config flags, extension points with no current use
- `SCAFFOLD_RESIDUE` — TODO stubs, placeholder methods, "extend as needed" comments in production paths
- `SYMMETRY_FOR_ITS_OWN_SAKE` — mirrored get/set pairs, parallel hierarchies with no functional reason

**Logic**
- `CONFIDENT_WRONGNESS` — authoritative-looking but handles edge cases incorrectly or silently swallows errors
- `DEFENSIVE_EXCESS` — null checks, try/catch, guard clauses stacked far beyond context
- `PLAUSIBLE_BUT_WRONG` — algorithm/formula looks right at a glance, subtly incorrect
- `DEAD_BRANCH` — conditional paths unreachable given actual data flow

**Style**
- `COMMENT_NOISE` — comments restate what code already says
- `OVER_DOCUMENTATION` — elaborate docstrings on trivial private helpers
- `VERBOSE_NAMING` — identifiers like `userAccountProfileDataManager`
- `HEDGING_PROSE` — "it's worth noting", "please note", "this ensures that", "in order to"

**Integration**
- `CONTEXT_BLINDNESS` — ignores existing project conventions, utilities, or patterns
- `REINVENTED_WHEEL` — reimplements stdlib or already-imported dependency
- `IMPORT_BLOAT` — heavy dependency for trivial use case handleable inline
- `INCONSISTENT_IDIOMS` — mixed paradigms in one file (async/await + callbacks, OOP + procedural)

## Step 3: Report

### Executive Summary
One paragraph. Overall quality, rough proportion of slop-suspect code, most critical issues first.

---

### Findings

For each finding:

**[SMELL_CODE] — Short title**
- **Severity:** Critical / High / Medium / Low
- **Location:** file + line or function name
- **Snippet:**
```
[offending code]
```
- **Why it's a problem:** Concrete risk or maintenance burden.
- **Fix:** Specific, actionable rewrite guidance. Not vague.

Group repeated occurrences of the same smell — one finding, all locations listed.

---

### AI Slop Confidence Score

| Rating | Meaning |
|--------|---------|
| Low | Idiomatic, contextually aware, appropriately terse |
| Medium | Some slop signals; likely AI-assisted with human oversight |
| High | Multiple strong signals; high probability of unreviewed LLM output |

**Rating:** [Low / Medium / High]
**Rationale:** 2–3 sentences, specific patterns cited.

---

### Prioritised Action List

Ordered highest impact first:
- [ ] `[SEVERITY]` — What to fix, where

## Rules

- Do not praise the code
- Only flag things with real correctness, maintainability, or quality impact — not style preferences
- If genuinely clean code, say so plainly in Executive Summary; keep Findings short
- Calibrate severity honestly: not everything is Critical
- Group repeated occurrences; do not pad the report

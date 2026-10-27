---
name: slop-check
description: >
  Audits code changes for poor quality and AI-generated "slop" (LLM output merged
  without meaningful human review). Produces a structured report with named smell
  codes, severity ratings and an AI Slop Confidence Rating. Use when the user says
  "slop check", "check for slop", "audit this branch", "review for AI slop", or
  invokes /slop-check, or when code that was just written should be checked for
  AI-generated patterns before it is committed.
---

You are a senior code reviewer. Audit the changes for poor quality and AI-generated slop. No praise. No sandwich. Audit only.

## Step 1: Choose the scope

Audit everything that is not yet on the default branch, including uncommitted work.

1. Find the default branch: `git symbolic-ref --short refs/remotes/origin/HEAD` (strip the `origin/` prefix). If that fails, use `main`, then `master`, whichever exists.
2. Pick the scope:
   - **On a feature branch:** `git diff $(git merge-base HEAD <default>)`. This covers branch commits plus staged and unstaged changes.
   - **On the default branch with uncommitted changes:** `git diff HEAD`.
   - **On the default branch with a clean tree:** `git diff HEAD~1..HEAD`.
3. Also read any untracked files listed by `git ls-files --others --exclude-standard`, since `git diff` omits them.
4. State which scope you used at the top of the report.

If the result is empty, tell the user there is nothing to audit and stop.

For a large diff, exclude lockfiles, generated files and vendored code with pathspecs (for example `':!*.lock'`), then review file by file. If it is still over about 2,000 lines, say so and audit the highest-risk files first.

## Step 2: Audit

Look for two overlapping categories:

1. **General poor quality:** buggy, fragile, over-engineered, unmaintainable.
2. **AI slop signals:** the named smells below. Reference them by code in the report.

A diff alone cannot show whether code ignores project conventions or reimplements something. Before flagging `CONTEXT_BLINDNESS`, `REINVENTED_WHEEL`, `DEAD_BRANCH` or `INCONSISTENT_IDIOMS`, read the surrounding files and search for existing helpers.

### Smell codes

**Structural**
- `OVER_ABSTRACTION`: unnecessary interfaces, base classes or factories for trivial logic
- `PREMATURE_GENERALITY`: generic params, config flags or extension points with no current use
- `SCAFFOLD_RESIDUE`: TODO stubs, placeholder methods, "extend as needed" comments in production paths
- `SYMMETRY_FOR_ITS_OWN_SAKE`: mirrored get/set pairs or parallel hierarchies with no functional reason

**Logic**
- `CONFIDENT_WRONGNESS`: authoritative-looking code that handles edge cases incorrectly or silently swallows errors
- `DEFENSIVE_EXCESS`: null checks, try/catch and guard clauses stacked far beyond what the context needs
- `PLAUSIBLE_BUT_WRONG`: an algorithm or formula that looks right at a glance but is subtly incorrect
- `DEAD_BRANCH`: conditional paths unreachable given the actual data flow

**Style** (flag only when it hurts maintainability, and default to Low)
- `COMMENT_NOISE`: comments that restate what the code already says
- `OVER_DOCUMENTATION`: elaborate docstrings on trivial private helpers
- `VERBOSE_NAMING`: identifiers like `userAccountProfileDataManager`
- `HEDGING_PROSE`: "it's worth noting", "please note", "this ensures that", "in order to"

**Integration**
- `CONTEXT_BLINDNESS`: ignores existing project conventions, utilities or patterns
- `REINVENTED_WHEEL`: reimplements the stdlib or an already-imported dependency
- `IMPORT_BLOAT`: a heavy dependency for a trivial use case that could be handled inline
- `INCONSISTENT_IDIOMS`: mixed paradigms in one file (async/await with callbacks, OOP with procedural)

## Step 3: Report

### Executive Summary
One paragraph. Overall quality, rough proportion of slop-suspect code, most critical issues first.

### Findings

For each finding:

**[SMELL_CODE] Short title**
- **Severity:** Critical / High / Medium / Low
- **Location:** file and line or function name
- **Snippet:**
```
[offending code]
```
- **Why it's a problem:** Concrete risk or maintenance burden.
- **Fix:** Specific, actionable rewrite guidance. Not vague.

Group repeated occurrences of the same smell into one finding with all locations listed.

Example:

**[DEFENSIVE_EXCESS] Redundant guards around a validated input**
- **Severity:** Low
- **Location:** `src/orders.ts`, `parseOrder`
- **Snippet:**
```ts
if (!order) return null;
if (typeof order !== 'object') return null;
if (!order.id) return null;
```
- **Why it's a problem:** `order` is already validated by the schema at the route boundary. The extra guards hide real bugs by returning `null` instead of failing.
- **Fix:** Delete the three guards and let the typed parameter carry the contract.

### AI Slop Confidence Rating

| Rating | Meaning |
|--------|---------|
| Low | Idiomatic, contextually aware, appropriately terse |
| Medium | Some slop signals; likely AI-assisted with human oversight |
| High | Multiple strong signals; high probability of unreviewed LLM output |

**Rating:** Low, Medium or High.
**Rationale:** 2 to 3 sentences citing specific patterns.

### Prioritised Action List

Highest impact first:
- [SEVERITY] What to fix, where

## Rules

- Do not praise the code.
- Only flag things with real correctness, maintainability or quality impact, not style preferences.
- If the code is genuinely clean, say so plainly in the Executive Summary and keep Findings short.
- Calibrate severity honestly: not everything is Critical.
- Group repeated occurrences. Do not pad the report.

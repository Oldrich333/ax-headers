# AX Header Specification

## Grammar (PCRE)

```
^# AX: (?P<tag>[A-Z][A-Z_]+) \| SUM: (?P<sum>[^|]+?) \| SIG: (?P<sig>\S+)\s*$
```

## Lifecycle gates

A tool that enforces AX headers should implement these five checks.

### AX-1 — presence (FAIL on miss)

Every `.py` file in the enforcement scope MUST begin with a line matching the
grammar above, with the following exemptions:

- Empty files (0 bytes)
- Files with `# noqa: AX` in the first 5 lines
- Files matching the project's exempt glob list (e.g. `tests/fixtures/*.py`,
  vendored code)
- `__init__.py` files with only imports and no executable logic

Missing header on a non-exempt file: FAIL.

### AX-2 — SIG bump on behavior change (WARN)

When a file has been modified in this diff, check whether its SIG changed.

- If behavior changed (signature, return type, new branch, new handler) and
  SIG did not bump: WARN.
- If SIG bumped but diff is whitespace/rename/comment only: WARN (wasteful
  bump confuses consumers).

Heuristic: `git diff` shows a function signature line or a `return` statement
line added/changed → SIG must bump.

### AX-3 — TAG in allowlist (FAIL on unknown tag)

Project must maintain an allowlist of valid TAG values. Any TAG not in the
allowlist: FAIL.

Rationale: unknown tags indicate either a typo (`PIEP` instead of `PIPE`) or
vocabulary drift (someone invented `HANDLER` without consensus). Either way
block and force explicit allowlist update.

### AX-4 — SUM not filler (WARN)

The SUM field must not be filler. Reject:

- Empty after `SUM:`
- Single word (e.g. `SUM: Stuff`)
- Generic verbs without object (`SUM: does things`, `SUM: handles`)
- Copy-pasted from another file (same exact SUM on 3+ files)

Heuristic: SUM must contain at least 3 non-stopword tokens and not duplicate
another file's SUM in the repo.

### AX-5 — multi-concern hint (WARN)

If SUM contains 2+ top-level concerns separated by " / " or " & " or " and ",
warn that the file may need splitting.

Example trigger: `SUM: Telegram connector / Slack connector / queue router`

## Tooling contract

A compliant enforcer consumes:

- A list of file paths (typically from `git diff --name-only`).
- Project config providing: allowlist of TAGs, exempt glob list, strict mode
  flag.

And emits:

- Per-file: PASS / WARN / FAIL with specific check ID and reason.
- Exit 0 if all PASS or all non-FAIL with strict mode off.
- Exit non-zero if any FAIL, or any WARN when strict mode on.

## Reference implementation

See `hooks/pre-commit-ax` for a minimal bash + awk pre-commit hook
implementing checks AX-1 and AX-3 (the two that are cheap + high-signal).
AX-2, AX-4, AX-5 require diff context or repo-wide awareness and are better
suited to a dedicated linter (`ax-lint`, to be released separately).

## Rationale for design choices

**Why one line, not a docstring?**
Docstrings are visible to humans but invisible to search-across-files tooling
without AST parsing. A `#`-prefixed line at file start is grep-friendly,
survives `head -n 1`, and adds zero load time.

**Why human-picked SIG, not hash?**
Hashes change on whitespace. SIG should mean "behavior version." Humans can
judge that; hashes cannot. The cost is one line of human discipline per
meaningful change — cheap compared to the consumer-side value.

**Why a closed TAG vocabulary?**
Open-ended tagging produces 200 half-synonyms within a year. A 15–20 item
allowlist enforced by the linter stays coherent at team scale.

**Why not Rust's `//! module doc` style?**
Python has no standard module-level metadata slot. Adding structured metadata
to the file-start comment is the most Python-idiomatic option — works in any
editor, any VCS, any grep tool.

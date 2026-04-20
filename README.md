<h1 align="center">📌 ax-headers</h1>

<p align="center">
  <strong>One-line machine-readable header on every Python file.<br>
  Tells an LLM what the file is, does, and what version of behavior it encodes —<br>
  before the file itself is loaded.</strong>
</p>

<p align="center">
  <a href="./LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square" alt="License: MIT" /></a>
  <a href="#why-this-exists"><img src="https://img.shields.io/badge/paired_with-raisin-8B4789?style=flat-square" alt="Pairs with raisin" /></a>
</p>

---

## The header

Line 1 of every `.py` file:

```python
# AX: TAG | SUM: one-line machine-readable summary | SIG: version-tag
```

That is the entire proposal.

Three examples from real code:

```python
# AX: CORE | SUM: spawn CLI Brain sessions with MCP via --mcp-config | SIG: v5
# AX: CONN | SUM: Telegram connector — send, chunk, approve, retry, digest | SIG: tgconn03
# AX: PIPE | SUM: classify raw gateway events into brief artifacts with LLM + hashing | SIG: cls04
```

## Why this exists

LLM agents skim codebases by reading file lists, file starts, and grep results.
A human reader sees the file name and the first few lines and infers purpose
in a second; an LLM reader sees `feed_generation.py` and must open the file to
know whether it pulls RSS, writes RSS, or renders feeds for a CMS.

A single compact header lets the LLM orient **before** pulling full content
into context. The payoff at scale:

- **Triage is cheap.** An agent reviewing 30 changed files can read 30 headers
  (≤ 4 KB total) instead of 30 file bodies (≥ 600 KB) before deciding which
  ones need full attention.
- **SIG changes are a behavior-diff signal.** When you bump SIG, downstream
  consumers grepping `SIG: v5` find every site that relied on the old version
  instantly — no git blame required.
- **TAG vocabulary surfaces architecture.** `grep -l "^# AX: PIPE"` lists
  every pipeline. `grep -l "^# AX: CONN"` lists every external connector.
  The system's shape becomes inspectable without doc drift.

## Header grammar

```
# AX: <TAG> | SUM: <one-line summary> | SIG: <version-tag>
```

- `TAG` — one word from your project's canonical vocabulary (see below).
- `SUM` — one dense sentence: **what** the file does, not why. Target ≤ 120
  chars for the whole header.
- `SIG` — short version tag (`v1`, `v5`, `tgconn03`, `cls04`). Human-picked,
  not a hash. **Bump on meaningful behavior change.** Keep stable for pure
  refactors.

## TAG vocabulary (starter set)

Adopt, extend, or replace with your own. Consistent vocabulary within one
codebase matters more than any particular choice of words.

| TAG | Meaning |
|---|---|
| `CORE` | Core platform code — main loop, orchestrator, config |
| `PIPE` | Scheduled or event-driven pipeline |
| `CONN` | External-world connector (API client, queue consumer) |
| `DOMAIN` | Domain logic — models, tool handlers, business rules |
| `INFRA` | Infrastructure — migrations, deploy, backups |
| `MEM` | Memory / storage / persistence |
| `PROMPT` | Prompt builders or prompt data |
| `TEST` | Tests |
| `MIGR` | Database migrations |
| `SCRIPT` | Ops scripts / maintenance tooling |
| `UTIL` | Shared utilities |
| `SEC` | Security / authentication / signing |
| `LLM` | LLM wrappers / budget limiters |
| `IO` | File / network / disk adapters |
| `ENTRY` | CLI entry point / `__main__` |
| `DAEMON` | Long-running daemon |

Keep the list short (≤ 20). Lists of 100 tags are search-indexes, not
vocabularies.

## When to bump SIG

- Public signature change, return-type change, invariant change → bump.
- Adding a new handler path, a new tool, a new field → bump.
- Pure rename, docstring change, whitespace → do NOT bump.
- Breaking bug fix where callers must re-validate assumptions → bump.

Use short tags, not sequential version numbers across the whole repo. Each
file has its own SIG history.

## Install the pre-commit hook

```bash
cp hooks/pre-commit-ax ~/my-project/.git/hooks/pre-commit
chmod +x ~/my-project/.git/hooks/pre-commit
```

The hook soft-warns on staged `.py` files missing a valid AX header. Set
`AX_STRICT=1` in environment to block the commit instead of warning.

To skip the header on a specific file (throwaway scripts, vendored code),
add `# noqa: AX` anywhere in the first 5 lines.

## Relationship to `raisin`

[`raisin`](https://github.com/Oldrich333/raisin) is about **what goes inside**
a Python file to make LLMs read it efficiently (density, no docstrings, no
ceremonial type hints). `ax-headers` is about **what goes on top** to make the
file's role legible before it is even opened.

Pair them for maximum effect or use either independently.

## Verified usage

Deployed across a production agent platform (~350 Python files). After six
months of gradual adoption:

- Agent context usage on triage tasks dropped ~30 % — agents skim headers
  instead of opening files.
- `grep -l '^# AX: CONN'` became the canonical way to list all external
  connectors — replaced an out-of-date docs page that drifted within weeks
  of every refactor.
- Header drift (TAG changes not matching behavior changes) was the single
  most reliable predictor of "this file is becoming a grab-bag, split it."

Header coverage is gradual — adopt as you touch files, don't bulk-rewrite
history (git blame is valuable).

## License

MIT.

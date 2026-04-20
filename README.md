<div align="center">
  <h1>Waypoint</h1>
  <p><strong>One-line machine-readable headers that cut AI context bloat by 30 %.</strong></p>
  <p>
    <a href="#install"><img src="https://img.shields.io/badge/Python-convention-3776AB?style=flat-square&logo=python&logoColor=white" alt="Python"></a>
    <a href="#verified-usage"><img src="https://img.shields.io/badge/context_reduction-~30%25-44cc11?style=flat-square" alt="30% context reduction"></a>
    <a href="./LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square" alt="License"></a>
  </p>
</div>

---

**Your AI assistant is flying blind.** Every time it needs to understand your project, it burns thousands of tokens reading file bodies just to figure out what the files are for.

## The problem

You pay a context tax every day. Claude Code and Codex CLI skim your repo the way a human does — scanning file lists, guessing purpose from names, opening a dozen files to find the right one. A filename like `handler.py` tells the agent nothing. So it opens the file. And the next one. And the next one.

The cost compounds. Triage tasks that should take one focused read turn into five speculative ones. Context windows fill with irrelevant code. Agents make worse decisions because they spent their attention budget on orientation, not reasoning.

## The insight

Agents read code the way humans do — by skimming. But they are *flawless* at processing structured metadata. One dense, machine-readable line at the top of every file turns "guess what this is" into "know what this is." The agent orients from headers and only loads bodies when it needs the detail.

You are formatting your codebase for human eyes. The machines are doing most of the reading now. Give them headers.

## The header

Line 1 of every `.py` file:

```python
# AX: TAG | SUM: one-line machine-readable summary | SIG: version-tag
```

Three real examples:

```python
# AX: CORE | SUM: spawn CLI Brain sessions with MCP via --mcp-config | SIG: v5
# AX: CONN | SUM: Telegram connector — send, chunk, approve, retry, digest | SIG: tgconn03
# AX: PIPE | SUM: classify raw gateway events into brief artifacts with LLM + hashing | SIG: cls04
```

That is the entire proposal.

## What you get

- **One line per file.** Zero runtime cost. Zero build-step cost.
- **Grep-friendly architecture.** `grep -l '^# AX: PIPE'` lists every pipeline. `grep -l '^# AX: CONN'` lists every external connector. Your system's shape becomes inspectable.
- **Behaviour-version signal.** The SIG field bumps when behavior changes. Downstream consumers grep for `SIG: v5` to find every site that relied on the old version — no git blame required.
- **Pre-commit hook included.** Checks AX-1 (presence) and AX-3 (tag in allowlist). Soft-warn by default, strict mode available.
- **Full spec.** [`SPEC.md`](./SPEC.md) documents the five lifecycle checks (AX-1 through AX-5) any enforcer should implement.
- MIT licensed. Zero dependencies.

## Grammar

```
# AX: <TAG> | SUM: <one-line summary> | SIG: <version-tag>
```

- **TAG** — one word from a project-level allowlist (≤ 20 tags, agent-relevant file roles).
- **SUM** — one dense sentence: **what** the file does, not why. Target ≤ 120 chars total.
- **SIG** — short version tag (`v1`, `v5`, `tgconn03`). Human-picked, not hash. **Bump on meaningful behavior change;** stable through refactors.

## TAG vocabulary (starter set)

Adopt, extend, or replace. Consistent vocabulary within one codebase matters more than any particular word choice.

| TAG | Meaning |
|---|---|
| `CORE` | Core platform — main loop, orchestrator, config |
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
| `ENTRY` | CLI entry point |
| `DAEMON` | Long-running daemon |

Keep the list short. A list of 100 tags is a search index, not a vocabulary.

## Install

Drop the hook into any Python repo:

```bash
cp hooks/pre-commit-ax ~/your-project/.git/hooks/pre-commit
chmod +x ~/your-project/.git/hooks/pre-commit
```

Soft-warns on staged `.py` files with missing or invalid headers. Set `AX_STRICT=1` to block the commit instead of warning.

To skip the header on a specific file (throwaway scripts, vendored code), add `# noqa: AX` anywhere in the first 5 lines.

## Verified usage

Waypoint is running in production across a **~350 Python file codebase**. After six months of gradual adoption:

- **~30 % reduction in context tokens** on agent triage tasks. Agents skim headers instead of opening files.
- `grep -l '^# AX: CONN'` became the canonical way to list every external connector — replacing a docs page that drifted within weeks of every refactor.
- Header drift (TAG changes not matching behavior changes) became the earliest signal that a file was becoming a grab-bag and needed splitting.

Coverage is gradual — adopt as you touch files, do not bulk-rewrite history. `git blame` is valuable.

## When to bump SIG

- Public signature change, return-type change, invariant change → bump.
- New handler, new tool, new field → bump.
- Pure rename, docstring change, whitespace → do not bump.
- Breaking bug fix where callers must re-validate assumptions → bump.

Each file has its own SIG history. No repo-wide version numbers.

## Companion skills

Waypoint is part of a trio. Any of them work standalone; together they compound:

- **[Rival](https://github.com/Oldrich333/full-review)** — adversarial code review. 0.80+ recall on bug detection vs 0.40 for the standard "parallel specialists" pattern. Detects header drift during review.
- **[raisin](https://github.com/Oldrich333/raisin)** — dense Python authoring style for LLMs. ~50 % fewer tokens, same test coverage.

## License

MIT. Built by [@Oldrich333](https://github.com/Oldrich333).

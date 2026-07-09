# ReadyBase

**Give your AI coding assistant the context a senior engineer already has, and catch it when what it writes drifts from how your team builds.**

ReadyBase is a local-first command-line tool that studies your repository once and answers, on every turn, the two questions a good reviewer asks: can AI work here safely, and does this change belong? It maps which files are load-bearing, what changed last week, who owns what, and what breaks if you touch a given file. Then it guards the risky edits, scores how ready the codebase is for AI-assisted work, and watches for vibe drift: AI output that compiles and passes review but doesn't fit your conventions.

One CGO-free static binary. Everything it generates lands in your repo as plain files you can read, diff, and commit. Nothing leaves your machine.

ReadyBase is a [Promptforce.ai](https://promptforce.ai) product. Promptforce.ai is a Day Two AI company.

> **Early Access.** Free while we launch, no obligation. Ships with a 60-day Pro trial that activates on first run, offline. Get your install link at [promptforce.ai](https://promptforce.ai).

## The problem

Your AI assistant writes code faster than it can understand yours. Understanding the codebase is the first thing a new engineer does, and the one thing the agent skips.

* **The confident stranger.** It edits like it read the whole repo. It didn't, and it can't see which file is load-bearing, so it changes the one with 40 dependents as casually as a typo fix.
* **Tribal knowledge doesn't travel.** "Don't touch that without asking Maria" lives in Slack and in Maria's head. The agent never got the memo.
* **Drift you only notice later.** Code that works but doesn't match your house style piles up one plausible PR at a time, until the codebase reads like a committee of strangers.
* **You find out in production.** A context-free edit looks clean in the diff. The blast radius shows up when the pager goes off.

The answer was never a smarter model. It is a foundation: map the codebase, guard the dangerous edits, and keep the AI coherent with how you build.

## Two axes

ReadyBase measures your codebase on two independent axes, both computed locally with no LLM calls.

* **Readiness** answers whether AI can work here safely. A 0 to 100 score, weighted by risk rather than size, across nine signals, with a ranked list of what to fix next.
* **Coherence** answers whether what the AI wrote belongs. ReadyBase learns your conventions from the code and git history and flags where new work has drifted from them.

Readiness gets AI in the door. Coherence keeps it on your rails.

## Install

```bash
# macOS / Linux. Download the script, review it, then run it.
curl -fsSL https://promptforce.ai/rb-install.sh -o rb-install.sh
less rb-install.sh
bash rb-install.sh
```

```powershell
# Windows PowerShell
iwr https://promptforce.ai/rb-install.ps1 -OutFile rb-install.ps1; .\rb-install.ps1
```

```bash
# Go toolchain, any platform
go install github.com/PromptForcePrime/readybase/cmd/readybase@latest
```

We don't do `curl | bash`. Download the script, read it, then decide. Already installed? Upgrade in place:

```bash
readybase update          # fetch the latest release, verify its checksum, replace in place
readybase update --check  # just report whether a newer version exists
```

## Quick start

```bash
readybase go .            # one command: scan, write guidance, index, set up
readybase score .         # AI-readiness score (0 to 100) with the fix list
readybase relate <file>   # what depends on this, what changed, what to test, before you edit
readybase serve           # open the readiness console at localhost:8765
readybase uninstall .     # remove everything ReadyBase generated (see also: hooks uninstall)
```

`readybase go .` is the canonical entry point. Everything it writes is plain text, committed to your repo, and reversible.

## What it does

Four moves, one binary.

### Map

A structural graph of your repo: direct and transitive dependents (blast radius), co-change partners mined from git history, ownership and bus factor, complexity hotspots, and a per-file intelligence card. Importance is ranked by graph centrality, so a file every dependency path routes through counts as load-bearing even if few files import it directly. Raw dependent-counts miss that.

```bash
readybase relate internal/storage/db.go   # blast radius, deps, and tests in one call
readybase story  internal/storage/db.go   # recent commits, authors, churn, bus factor
readybase explore "users get a 500 on password reset"   # where to start for a task or bug
```

### Guard

ReadyBase writes the guidance your assistant reads (`CLAUDE.md`, `AGENTS.md`, guard rules) and installs optional git hooks. When your AI is about to edit a high-blast-radius file, the warning arrives at the right moment. No prompt engineering, fail-open by default, and the guidance travels with the repo.

### Watch

ReadyBase learns your naming, structure, and error-handling conventions by consensus, not from a style guide nobody reads, and flags where AI-written code has strayed. Drift caught in the moment keeps the codebase feeling like one author.

### Govern

One readiness score with a ranked list of fixes ordered by points-per-hour, so the cheapest wins come first. Re-scan any time; `readybase diff .` proves the number moved. A manager-facing summary and portfolio ranking (`readybase portfolio <dir>`) roll many repos into one view.

## The AI-readiness score

One number from 0 to 100, composed from nine signals, weighted by risk rather than size so a large well-owned repo isn't punished and a fragile one can't hide. It is computed from your repo on disk, and every signal is inspectable.

| Signal | Weight | What it measures |
|--------|:------:|-----------------|
| Test coverage | 17 | Share of code reached by tests |
| Test quality | 3 | Assertion density, real probes vs smoke |
| Documentation | 15 | README freshness, docs, context files |
| Dependency health | 15 | Outdated dependencies, known advisories |
| Bus factor | 15 | How many people actually know each area |
| Build complexity | 10 | Env vars, containerization, reproducibility |
| Code complexity | 10 | File and function size, nesting |
| CI/CD maturity | 10 | Tests in CI, lint, deploy automation |
| Structure | 5 | Package modularity, directory depth, cross-package coupling |

`readybase score .` prints the full breakdown; `readybase diff .` compares it to the last scan. The coherence axis is tracked alongside it and surfaced in the console and reports.

## Connect it to your assistant

ReadyBase computes context locally and hands it to whatever assistant you already use.

| Surface | How to connect |
|---------|----------------|
| CLI-backed skills (default) | `readybase go .` installs lightweight skills so `explore`, `relate`, and `story` work with no always-on MCP server, so there is zero idle token cost |
| MCP (opt-in) | `readybase mcp` registers the tools: `readybase_explore` to start, `readybase_relate` for blast radius, deps, and tests, `readybase_story` for history and authors, plus the full set including `score`, `coverage`, `convention`, `who_owns`, `ghost`, and `succession` |
| Git hooks | `readybase hooks install` adds a blast-radius warning on commit, a readiness gate on push, and session context on start |
| Editors (LSP) | `readybase lsp` |
| Dashboard | `readybase serve`, then open `http://localhost:8765` |
| CI (GitHub Action) | gate a build on readiness in three lines (below) |

Gate a pull request on AI-readiness:

```yaml
# .github/workflows/readiness.yml
- uses: PromptForcePrime/readybase@v1
  with:
    path: .
    min-score: 50
    fail-under: true
```

The action emits `score`, `grade`, `ai-readiness`, and `passed`, and writes a job summary. Add `sarif: true` (with `permissions: security-events: write`) to surface findings in the Security tab and as inline PR annotations. `readybase export` also emits scorecards for Backstage, Port.io, and Cortex.

## Why ReadyBase

The category is full of tools that retrieve your code for an agent. ReadyBase is different where it counts.

* **It judges, not just retrieves.** A readiness score, a coherence verdict, and a guard that gates the risky edit, not only a search index. It tells the AI whether a change belongs, not just what is in the file.
* **Network-free, provably.** No telemetry, no license phone-home during analysis, no upload. Your source never leaves the machine, a claim you can verify with `tcpdump` (below).
* **Cost-first.** CLI-backed skills by default, so there is no always-on MCP server burning tokens on every session. Context is budgeted to the significant minority, not dumped wholesale.
* **Deterministic and inspectable.** Same commit in, same result out. Everything generated is plain text you can diff, commit, or remove.

## Principles

1. **Local-first.** All analysis runs on the repo on disk. ReadyBase reads your files; it does not send them anywhere. See [`PRIVACY.md`](PRIVACY.md).
2. **Single binary.** One CGO-free static executable. No runtime, no background service phoning home, no sidecar to supervise.
3. **Inspectable output.** Everything ReadyBase generates lands in your repo as plain text you can read, diff, and commit, or remove with `readybase uninstall`.

Verify it yourself:

```bash
# ReadyBase makes no outbound calls during a scan. This should print nothing.
sudo tcpdump -i any -n 'dst port 443' 2>/dev/null
# ...now run `readybase scan .` in another terminal.
```

The one deliberate exception is `readybase update`, which fetches a new release from GitHub only when you run it.

## Credits and attribution

ReadyBase is a distillery. We study the best ideas in code intelligence and re-author the algorithm natively in Go, inside our own principles. Copyright protects code, not ideas, so for the projects and research below, no source code is copied into ReadyBase. These are courtesy credits, not license obligations. We name them because the ideas are theirs and the credit should be too.

### Ideas distilled into native Go (clean-room, no code used)

| Source | The idea we learned from | License |
|--------|--------------------------|---------|
| [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) | A static dependency graph you can write rules against | MIT |
| [madge](https://github.com/pahen/madge) | Module graph with circular-dependency detection | MIT |
| CodeScene / *Your Code as a Crime Scene* (Adam Tornhill) | Behavioral code analysis: hotspots, change coupling, knowledge maps | proprietary (book) |
| Truck Factor research (Avelino, Passos, Hora, Valente, 2016) | Estimating bus / truck factor from version-control authorship | research |
| Community detection (Blondel et al., *Louvain*) and centrality (Brandes, *betweenness*) | Finding architectural modules and load-bearing files in a graph | research |
| [GitHub CODEOWNERS](https://docs.github.com/articles/about-code-owners) | Mapping paths to the people and teams who own them | convention |
| [SonarQube](https://github.com/SonarSource/sonarqube) | A composite quality gate over many independent signals | LGPL-3.0 |
| [agents.md](https://agents.md) / Anthropic Claude Code | Project context files an agent reads (`AGENTS.md`, `CLAUDE.md`) | convention / platform |

### Components we actually package (real license obligations)

ReadyBase links a small set of third-party Go modules into the released binary. The structural parser is built on the tree-sitter idea via a pure-Go binding; the rest are storage, formatting, and platform glue.

| Component | Role | License |
|-----------|------|---------|
| [gotreesitter](https://github.com/odvcencio/gotreesitter) | Pure-Go incremental parsing to AST (implements the [tree-sitter](https://github.com/tree-sitter/tree-sitter) idea) | MIT |
| [modernc.org/sqlite](https://gitlab.com/cznic/sqlite) | Embedded database for scans, history, and the console | BSD-3-Clause |
| [google/uuid](https://github.com/google/uuid) | Identifiers for scans and records | BSD-3-Clause |

The complete set of third-party Go modules compiled into the binary is in [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md), with the full verbatim text of every license under [`LICENSES/`](LICENSES/); see also [`NOTICE`](NOTICE).

If we have miscredited or mislabeled anything, please open an issue and we will correct it.

## License and terms

ReadyBase is distributed as a binary, in Early Access, provided as-is with no warranty. Early Access is free while we launch; Pro and Team plans are available on request at [contact@daytwoai.com](mailto:contact@daytwoai.com).

Third-party components compiled into the binary retain their own licenses, recorded in [`NOTICE`](NOTICE) and [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md).

## A note on Anthropic

Day Two AI is an Anthropic partner. **ReadyBase is built independently, with no Anthropic participation, funding, or endorsement.** It is not an Anthropic product and is not affiliated with Anthropic. "Claude" and "Anthropic" are trademarks of Anthropic; they are used here only to describe interoperability. ReadyBase analyzes your code locally and hands context to whatever assistant you already use; it never sends your source anywhere.

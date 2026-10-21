# ReadyBase

**Give your AI coding assistant the context a senior engineer already has, catch the edits that don't belong, and know how far the agent can be trusted to run on its own.**

ReadyBase is a local-first command-line tool. It studies your repository once: which files are load-bearing, what changed recently, who owns what, and what breaks if you touch a file. It then hands that context to your assistant on every turn, guards the risky edits, and scores how ready the codebase is for AI-assisted work.

One CGO-free static binary. What it writes lands in your repo as plain files you can read, diff, commit, or remove. Analysis runs on your machine and sends nothing anywhere.

ReadyBase is a [Promptforce.ai](https://promptforce.ai) product. Promptforce.ai is a Day Two AI company.

> **Early Access.** Ships with a 45-day Pro trial that activates on first run, offline. Get your install link at [promptforce.ai](https://promptforce.ai).

## Install

```bash
# macOS / Linux: download the script, read it, then run it
curl -fsSL https://promptforce.ai/rb-install.sh -o rb-install.sh
less rb-install.sh
bash rb-install.sh
```

```powershell
# Windows PowerShell
iwr https://promptforce.ai/rb-install.ps1 -OutFile rb-install.ps1; .\rb-install.ps1
```

Releases are published here, and Windows binaries are signed. To upgrade in place, run `readybase update`. To only check for a newer version, run `readybase update --check`.

## Quick start

```bash
readybase go .              # scan, index, write guidance, install hooks
readybase score .           # readiness score (0 to 100) and what to fix first
readybase relate <file>     # before an edit: dependents, co-change, tests, owners
readybase symbol <name>     # where a name is defined, from the index
readybase loop-audit .      # how far an agent may run unsupervised (L0 to L3)
readybase uninstall .       # remove what ReadyBase wrote; your own edits are kept
```

## What it does

All analysis is deterministic: no LLM calls, and the same commit always gives the same result.

**Map.** ReadyBase builds a structural graph of the repo:

- direct and transitive dependents (blast radius);
- co-change partners mined from git history;
- ownership and bus factor;
- complexity hotspots.

Load-bearing files are ranked by graph centrality, not raw import counts. Four commands use the map:

- `relate` gives a file's context in one call, with `--skeleton` for a symbol outline and paged sections for large files.
- `symbol` finds a definition without guessing a directory.
- `explore "<task, bug or stacktrace>"` narrows a task, bug or stacktrace to where to start.
- `story <file>` tells a file's history.

**Guard.** ReadyBase writes the guidance your assistant reads (`CLAUDE.md`, `AGENTS.md`, guard rules) and installs hooks that warn at the moment an agent is about to edit a high-blast-radius file. Hooks fail open, so ReadyBase never breaks a session. Two commands manage clearance:

- `readybase check <file>` says whether an agent edit is cleared.
- `readybase approve <file> --ttl 2h` clears one file for a bounded time.

**Watch.** ReadyBase learns your naming, structure and error-handling conventions by consensus, then flags code that has drifted from them (`readybase drift .`). Before an edit, `readybase suggestion <file>` lists:

- the conventions that apply;
- code that already exists and could be reused;
- the blast radius.

**Score.** One readiness number with a fix list ordered by points per hour. `readybase portfolio <dir>` ranks many repos.

**Certify.** Loop Readiness sets how much autonomy the repo has earned, from L0 to L3, and `readybase gate . --emit` writes `gate.yaml`. That file is the list of files an agent must not touch without human review. It is derived from the dependency graph and from authorship provenance (whether a person or an AI wrote the code). A circuit breaker stops an agent that keeps repeating the same failure.

**Review gate.** `readybase review-gate enable` posts a `readybase/agent-review` status on your open pull requests. The status fails while any file an agent changed in the PR is load-bearing and nobody has marked it reviewed, and it passes once someone has. When ReadyBase cannot decide, the status is left pending, never passed. It is off by default and set per repo.

**Turn record.** Hooks record what each agent turn changed, as file and commit counts and never file contents. ReadyBase then checks whether those changes were still there 7, 30, 90 and 180 days later. `readybase events` reports this record, and `readybase recording off` pauses it.

## The readiness score

| Signal | Weight | What it measures |
|--------|:------:|------------------|
| Test coverage | 17 | Share of code reached by tests |
| Bus factor | 15 | How many people know each area |
| Documentation | 14 | README freshness, docs, context files |
| Dependency health | 14 | Outdated dependencies, known advisories |
| Build complexity | 10 | Env vars, containerization, reproducibility |
| Code complexity | 10 | File and function size, nesting |
| CI/CD maturity | 10 | Tests in CI, lint, deploy automation |
| Structure | 5 | Modularity, depth, cross-package coupling |
| Security | 5 | Credential-shaped strings, a disclosure policy |

`readybase score .` prints the breakdown and `readybase diff .` compares it with the last scan.

Loop Readiness is a level, set by the weakest blocking gate so that one gap cannot be averaged away:

- **L0, not ready:** every agent change needs review.
- **L1, report only:** agents propose and every change is reviewed.
- **L2, supervised:** agents may merge outside `gate.yaml`.
- **L3, autonomous:** the gate and the circuit breaker are the safety net.

## Connect it to your assistant

| Surface | How |
|---------|-----|
| Skills (default) | `readybase go .` installs CLI-backed skills for `explore`, `relate` and `story`. No always-on server, so there is no idle token cost |
| MCP (opt-in) | `readybase setup . --profile mcp` registers the tool set (explore, relate, story, score, ownership and more) |
| Git hooks | `readybase hooks install`: blast-radius warning on commit, readiness gate on push |
| Editors | `readybase lsp` |
| Prefex | `readybase serve` answers Prefex's versioned API on port 8018; `readybase serve --ui` adds the local dashboard |
| Exports | `readybase export` writes a scorecard for Backstage, Port.io or Cortex, or SARIF for GitHub code scanning |

## Optional judgment layer: Laya or Jev

Some calls need a judgment that the graph alone cannot make, such as:

- whether a file looks security-sensitive;
- how to rank context for a task;
- what kind of request started a recorded turn.

For these, ReadyBase can ask a model. Its answers are advisory only: they never block an edit or change a verdict. You have two options:

- **Laya, on your machine.** `readybase sidecar install` (or `readybase go . --with-laya`) installs [Laya](https://huggingface.co/convaiinnovations/laya) in a local Python environment and runs it as a sidecar on localhost. Questions and prompt text stay on the machine. `readybase sidecar status` shows what it has answered.
- **Jev, hosted, bring your own key.** Export `TYPESAFE_API_KEY` to use TypeSafe's Jev. What it receives depends on the call:
  - the security advisory sends the first 6,000 characters of the file it checks;
  - context ranking sends the task and file names;
  - turn labels send a turn's structure, never prompt text.

  Use Laya if code must not leave the machine.

With neither installed, the layer is off.

## Privacy

Analysis reads the repo on disk and sends nothing anywhere. License checks are offline. ReadyBase makes outbound calls only in these cases, each one something you turn on:

| Call | When |
|------|------|
| GitHub releases | You run `readybase update` |
| GitHub API (commit statuses) | You enable `review-gate`, using your `gh` login or `READYBASE_GITHUB_TOKEN` |
| TypeSafe API | You set `TYPESAFE_API_KEY` (the security advisory includes the checked file's content) |
| Your webhook URL | You configure one |

Laya and Prefex run on localhost. You can check the no-network claim yourself: run `sudo tcpdump -i any -n 'dst port 443'` while `readybase scan .` runs in another terminal, and it should print nothing. See [`PRIVACY.md`](PRIVACY.md).

## Credits

ReadyBase re-authors good ideas natively in Go. **No source code from the projects below is copied into ReadyBase**, so these are courtesy credits, not license obligations, and their licenses do not apply to ReadyBase.

| Source | Idea |
|--------|------|
| [dependency-cruiser](https://github.com/sverweij/dependency-cruiser), [madge](https://github.com/pahen/madge) | Dependency graphs with rules and cycle detection |
| CodeScene / *Your Code as a Crime Scene* (Adam Tornhill) | Hotspots, change coupling, knowledge maps |
| Truck Factor research (Avelino, Passos, Hora, Valente, 2016) | Bus factor from authorship |
| Louvain (Blondel et al.), betweenness (Brandes) | Modules and load-bearing files in a graph |
| [GitHub CODEOWNERS](https://docs.github.com/articles/about-code-owners) | Mapping paths to owners |
| [SonarQube](https://github.com/SonarSource/sonarqube) | A composite quality gate over many signals |
| [agents.md](https://agents.md) / Claude Code | Context files an agent reads |
| [CodeGraph](https://github.com/colbymchenry/codegraph) | A symbol's source returned inline from a pre-built index |
| [Serena](https://github.com/oraios/serena) | Symbol outlines and reference navigation |
| [grepai](https://github.com/yoanbernabeu/grepai) | Context as a token-budget problem |
| [Repomix](https://github.com/yamadashy/repomix) | Signature compression under a token budget |
| [SCIP](https://github.com/sourcegraph/scip) | Canonical symbol IDs, per-document indexing |
| [VibeDrift](https://github.com/VibeDrift/VibeDrift) | Consensus pattern inference, near-duplicate detection |
| [loop-engineering](https://github.com/cobusgreyling/loop-engineering) | L0 to L3 levels, `gate.yaml`, the circuit breaker |
| [Factory.ai](https://factory.ai) Agent Readiness | A gated maturity ladder over a score |
| Kenogami readiness | Level set by the weakest dimension |
| [Memtrace](https://github.com/syncable-dev/memtrace-public) | Fan-in-weighted impact and significance budgeting |
| Repowise | Validating a signal against later bug-fix commits |

**Compiled into the binary.** The binary includes 11 Go modules under MIT or BSD-3-Clause, among them [gotreesitter](https://github.com/odvcencio/gotreesitter) for parsing and [modernc.org/sqlite](https://gitlab.com/cznic/sqlite) for storage. They are listed with versions in [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md), with the full license texts in [`LICENSES/`](LICENSES/) and a summary in [`NOTICE`](NOTICE).

**Installed only on request, not bundled.** `readybase sidecar install` fetches [Laya](https://huggingface.co/convaiinnovations/laya) (Apache-2.0) and Flask (BSD-3-Clause) from PyPI into `~/.readybase/laya-venv`.

If we have miscredited anything, please open an issue.

## License and Anthropic

ReadyBase is distributed as a binary, in Early Access, as-is and with no warranty. For Pro and Team plans, contact [[PII_EMAIL_2]](mailto:[PII_EMAIL_2]).

Day Two AI is an Anthropic partner. **ReadyBase is built independently, with no Anthropic participation, funding or endorsement.** "Claude" and "Anthropic" are trademarks of Anthropic, used here only to describe interoperability.

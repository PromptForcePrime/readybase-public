# ReadyBase

**Make your codebase ready for AI agents, with a score that says how ready.**

ReadyBase reads a repository on disk and scores how safely an AI coding agent can work in it. It builds a
dependency and co-change graph, finds the files an edit will ripple into, generates adversarial tests that
surface real bugs, and writes the context files (`CLAUDE.md`, `AGENTS.md`, guard rules) an agent reads
before it touches your code. One CGO-free Go binary, no source leaving your machine.

ReadyBase is a [PromptForce.ai](https://promptforce.ai) product. PromptForce.ai is a Day Two AI company.
It is the context plane to [Prefex](https://promptforce.ai)'s cost plane.

> ReadyBase is in **Early Access**: free for 60 days, no obligation. Get your install link at
> [promptforce.ai](https://promptforce.ai).

---

## Install

```bash
# macOS / Linux. Download the script and review it before running.
curl -fsSL https://promptforce.ai/rb-install.sh -o rb-install.sh
less rb-install.sh
bash rb-install.sh
```

```powershell
# Windows PowerShell
iwr https://promptforce.ai/rb-install.ps1 -OutFile rb-install.ps1; .\rb-install.ps1
```

We do not do `curl | bash`. Download the script, read it, then decide. You can also grab a binary for your
platform directly from [Releases](https://github.com/PromptForcePrime/readybase-public/releases).

ReadyBase is a single CGO-free static binary, no runtime dependencies.

```bash
readybase scan .          # Score your codebase (0-100)
readybase go .            # Scan + setup + index + probe in one pass
readybase serve           # Dashboard at localhost:8765
```

---

## What it does

**A 9-signal AI-readiness score (0-100).** One composite number over nine independent signals, each
explainable on its own.

| Signal | Weight | What it measures |
|--------|--------|------------------|
| Test coverage | 17 | Share of code exercised by tests |
| Test quality | 3 | Assertion density, probe tests |
| Documentation | 15 | README freshness, docs, agent context files |
| Dependency health | 15 | Outdated dependencies, known CVEs |
| Bus factor | 15 | How concentrated knowledge is across contributors |
| Build complexity | 10 | Env vars, Docker, CI surface |
| Code complexity | 10 | File and function size, nesting |
| CI/CD maturity | 10 | Tests in CI, lint, deploy |
| Structure | 5 | Package count, directory depth |

**Blast radius before you edit.** `readybase blast <file>` lists every dependent and co-change partner of a
file, so an agent (or a person) sees what a change ripples into before making it.

**Adversarial probe testing.** Generates grey-box and chaos tests that find real bugs: nil and boundary
inputs, panic recovery, concurrent access, context cancellation, and seam tests between components.

**Agent instrumentation.** Logs every MCP tool call and reports whether an agent actually followed the
guard rules: "you edited 4 guarded files, called blast on 0."

Each signal and check is independent. ReadyBase reports what it can compute and is explicit about what it
cannot.

---

## Commands

| Command | What it does |
|---------|--------------|
| `readybase scan <path>` | Analyze a codebase, compute the AI-readiness score |
| `readybase go <path>` | Run scan + setup + index + probe |
| `readybase probe <path>` | Generate adversarial tests (`--run` to execute) |
| `readybase index <path>` | Build per-file intelligence cards |
| `readybase blast <file>` | Show all dependents before editing |
| `readybase setup <path>` | Generate `CLAUDE.md`, `AGENTS.md`, guard rules |
| `readybase diff <path>` | Compare current vs previous scan |
| `readybase score <path>` | Show the score breakdown |
| `readybase portfolio <dir>` | Score multiple repos |
| `readybase serve` | Launch the dashboard at `:8765` |
| `readybase mcp` | Start the MCP server for Claude Code |
| `readybase usage <path>` | Agent tool usage + guard compliance report |

## MCP tools (Claude Code)

Configured as an MCP server, ReadyBase gives Claude Code:

- `readybase_score` - AI-readiness score and per-signal breakdown
- `readybase_blast` - blast radius for a file (dependents, co-changes)
- `readybase_deps` - dependency analysis
- `readybase_cochange` - co-change partners
- `readybase_coverage` - test coverage
- `readybase_file_context` - per-file intelligence card

---

## Principles

1. **Local-first.** ReadyBase reads the repository on disk and sends no source code anywhere. No telemetry,
   no analytics, no external logging.
2. **Explainable, not a black box.** Every signal in the score is computed from something you can inspect,
   and the breakdown shows its contribution.
3. **Binary, not source.** One CGO-free Go binary. Nothing to supervise, no sidecar.

---

## Credits and attribution

ReadyBase is a distillery: we study the best ideas in code analysis and re-author the *algorithm* natively
in Go, inside our own principles. Copyright protects code, not ideas, so for the projects below **no source
code is copied into ReadyBase**: these are courtesy credits, not license obligations.

| Source | The idea we learned from |
|--------|--------------------------|
| [dependency-cruiser](https://github.com/sverweij/dependency-cruiser) | Static dependency graph with rules written against it |
| [madge](https://github.com/pahen/madge) | Module dependency graph and circular-dependency detection |
| CodeScene / "Your Code as a Crime Scene" (Adam Tornhill) | Behavioral code analysis: hotspots, change coupling, knowledge maps |
| Truck Factor research (Avelino, Passos, Hora, Valente, 2016) | Estimating bus / truck factor from version-control authorship |
| [GitHub CODEOWNERS](https://docs.github.com/articles/about-code-owners) | Mapping paths to the people and teams who own them |
| [SonarQube](https://github.com/SonarSource/sonarqube) | A composite quality gate over many independent signals |
| [agents.md](https://agents.md) / Claude Code | Project context files an agent reads (`AGENTS.md`, `CLAUDE.md`) |

The Go libraries actually compiled into the released binary, and their licenses, are listed in
[`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md), with full verbatim license texts under
[`LICENSES/`](LICENSES/). See also [`NOTICE`](NOTICE).

If we have miscredited or mislabeled anything, please open an issue and we will correct it.

---

## License and terms

ReadyBase is distributed as a binary, in Early Access, provided as-is with no warranty. Source code is not
included in this repository (binary releases only). Early Access is free for 60 days; Pro and Team plans
are available on request at [contact@daytwoai.com](mailto:contact@daytwoai.com).

Third-party Go modules compiled into the binary retain their own licenses, recorded in
[`NOTICE`](NOTICE) and [`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md).

---

## A note on Anthropic

Day Two AI is an Anthropic partner. **ReadyBase is built independently, with no Anthropic participation,
funding, or endorsement.** It is not an Anthropic product and is not affiliated with Anthropic. "Claude"
and "Anthropic" are trademarks of Anthropic; they are used here only to describe interoperability.

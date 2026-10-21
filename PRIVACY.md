# Privacy: your code never leaves your machine

ReadyBase analyzes your codebase locally, on your own hardware, using your own git history. No code is
uploaded. No server sees your source. No third party is in the loop.

## Principles

- **Local-first.** All analysis runs on the repo on disk. The binary reads your files; it does not send
  them anywhere.
- **Single binary.** One static, CGO-free executable. No runtime, no background service phoning home, no
  dependencies to audit.
- **Inspectable output.** Everything ReadyBase generates lands in your repo as plain files you can read,
  diff, and commit, or remove.

## What stays on your machine

| Data | Leaves your machine? |
|------|----------------------|
| Your source code | Never |
| Git history and authorship | Never |
| Readiness scores and reports | Never (stored in your repo, under `.readybase/`) |
| Dependency graph and blast radius | Never |
| License validation | Never (checked offline) |

## Common questions

**Does ReadyBase send my code to an AI model?**
Not by default. ReadyBase computes context locally and hands it to whatever AI assistant *you* already
use. The optional judgment layer can run on your machine (Laya, `readybase sidecar install`) or on
TypeSafe's hosted Jev if you set `TYPESAFE_API_KEY`. With Jev, the security advisory sends the first
6,000 characters of the file it checks; use Laya if code must not leave the machine.

**What does ReadyBase ever send over the network?**
Only what you turn on: `readybase update` (GitHub releases), `readybase review-gate` (commit statuses
via the GitHub API), Jev (with your key), and webhooks you configure. Laya and the Prefex API run on
localhost.

**What gets written to my repo?**
Guidance files (`CLAUDE.md`, `AGENTS.md`), guard rules, and a local intelligence index, all plain text,
all yours. Remove them anytime with `readybase uninstall`.

**Can I run it fully offline?**
Yes. Analysis and license checks need no network.

**Is it safe for proprietary or regulated codebases?**
That is exactly who local-first is for. Nothing about your code, structure, or contributors is
transmitted, so there is no third-party data-processing agreement to negotiate.

## Verify it yourself

```bash
# ReadyBase makes no outbound calls during a scan. With a scan running, this should stay quiet.
sudo tcpdump -i any -n 'dst port 443' 2>/dev/null
```

Questions: [contact@daytwoai.com](mailto:contact@daytwoai.com).

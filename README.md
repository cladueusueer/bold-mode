# bold-mode

**Risk is allowed in the idea, not in the blast radius.**

A Claude Code plugin that lets Claude try the *bold* idea, the rewrite, the deleted layer, the "what if this didn't exist", without putting your working code at risk. Every experiment runs in an isolated copy, inside a hard step budget, and ends with an honest scoreboard against your baseline.

[![tests](https://github.com/cladueusueer/bold-mode/actions/workflows/test.yml/badge.svg)](https://github.com/cladueusueer/bold-mode/actions/workflows/test.yml)
[![license: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-ffb020.svg)](#install)
[![good first issues](https://img.shields.io/github/issues/cladueusueer/bold-mode/good%20first%20issue?label=good%20first%20issues&color=5fd38d)](https://github.com/cladueusueer/bold-mode/labels/good%20first%20issue)

[עברית](README.he.md) · [Español](README.es.md)

![An illustrative bold-mode run: isolate, baseline, build, measure, report](assets/demo.svg)

<sub>Illustrative example of the report format, not a measurement from a real project. See [examples/sample-report.md](examples/sample-report.md).</sub>

---

## Why

Ask an AI to improve something and you usually get the safe, incremental answer. That's the right default. But sometimes the real win is the uncomfortable idea, and trying it costs time and risks a mess.

bold-mode makes the bold attempt **cheap and safe**:

| Without bold-mode | With bold-mode |
|---|---|
| Safe, incremental suggestions | Safe, Bold *and* Wild ideas, built and measured |
| "This looks cleaner" | A table of numbers vs. your baseline |
| Big rewrites in your working tree | Work in a git worktree or `/tmp` copy only |
| Experiments that never end | A hard step budget, enforced by a hook |
| The bold idea "always wins" | Honest reports, including "Bold lost" |

## What's inside

- **`bold-mode` skill**: the method. Isolate, baseline, diverge into Safe / Bold / Wild, build, measure, report, hand over.
- **11 lenses**: short reframing playbooks (delete a layer, invert the flow, data over code, ten-x, …) that Claude reads to come up with ideas that aren't the obvious one. [Browse them](skills/bold-mode/lenses/README.md).
- **Guard hook**: active *only* during a bold run. It:
  - counts steps and **stops the run** when the budget is used up (default 25),
  - **blocks edits to your original files** (work must happen in the isolated copy),
  - **blocks force-push, pushing to main/master/release, publishing, deploying, infra changes and dropping data**,
  - blocks `git reset --hard`, `git clean`, `git merge` and recursive deletes inside the original.
- **`/bold` command**: start a run in one line.

## Install

In Claude Code:

```
/plugin marketplace add cladueusueer/bold-mode
/plugin install bold-mode@bold-mode
```

Restart Claude Code (or start a new session) so the hooks load. Requires `python3` on your PATH for the guard; without it, the skill still works but nothing is enforced.

**Skill only** (no hook, works in Claude Code, claude.ai and other agents that read `SKILL.md`): copy `skills/bold-mode/` into `~/.claude/skills/bold-mode/`, or zip that folder and upload it as a skill in claude.ai.

## Use

```
/bold the caching layer in src/cache — it's 900 lines and still slow
```

or just say it:

```
Be bold with the onboarding flow. Use bold-mode.
```

You get back something like this (illustrative):

| | Baseline | Bold: delete the cache layer, use HTTP caching | Safe: tune TTLs | Wild: precompute at build |
|---|---|---|---|---|
| Checks passing | 42/42 | 42/42 | not built (estimate: 42/42) | 39/42 |
| Lines | 912 | 140 | ~900 | 210 |
| p95 latency | 180 ms | 95 ms | ~160 ms | 20 ms |
| Risk to adopt | – | medium: depends on CDN config | low | high: stale data |
| Effort to adopt | – | half a day | an hour | two days |

**Recommendation:** adopt Bold. Result in branch `bold/cache` (worktree `../app-bold-cache`).

See [examples/sample-report.md](examples/sample-report.md) for a full example.

## When *not* to use it

Small fixes, bug fixes, typos, routine refactors, anything stable. The skill says so explicitly, and only triggers on a clear request.

## Settings

| Variable | Default | Effect |
|---|---|---|
| `BOLD_MODE_BUDGET` | `25` | Max counted steps (Bash / Edit / Write) per run. Reads are free. |
| `BOLD_MODE_GUARD` | `on` | Set `off` to disable all blocking (budget and blast radius). |

## How the guard knows a run is active

A run starts when the `bold-mode` skill is invoked or a prompt starts with `/bold` (or mentions `bold-mode`). It ends when Claude finishes its turn. State lives in your temp folder, per session. Outside a run the hook does nothing.

## Honest limits

- The guard sees tool calls, not intentions. A Bash command that writes into the original with a redirect (`>`), or a script that does it, is not caught. The skill instructions and the isolated copy are the main protection; the hook is the seatbelt.
- Command patterns are a blocklist, not a sandbox. If you need hard guarantees, run Claude Code in a container.
- "Measured" is only as good as your tests. With no tests, the skill writes a few quick checks and says the comparison is weaker.

## Contributing

New here? Pick a [good first issue](https://github.com/cladueusueer/bold-mode/labels/good%20first%20issue): most are "write one lens" (one short markdown file).

The easiest contribution is a **new lens**: one short markdown file with a reframing move that has helped you find a better solution. See [CONTRIBUTING.md](CONTRIBUTING.md). Guard patterns, translations and real-world reports are welcome too.

## License

MIT

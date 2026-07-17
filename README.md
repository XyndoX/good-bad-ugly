<p align="center">
  <img src="assets/logo.svg" width="180" alt="good-bad-ugly logo">
</p>

<h1 align="center">good-bad-ugly</h1>

<p align="center">
  <em>Three gunslingers, three model tiers. You are The Good.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude%20Code-plugin-1f1f1f?style=flat-square" alt="Claude Code plugin">
  <img src="https://img.shields.io/badge/tiers-good%20·%20bad%20·%20ugly-c98a2e?style=flat-square" alt="Three tiers">
  <img src="https://img.shields.io/github/license/XyndoX/good-bad-ugly?style=flat-square&color=1f1f1f" alt="MIT license">
</p>

---

A [Claude Code](https://claude.com/claude-code) plugin that turns one session
into a **tiered multi-agent orchestrator**. You stay the brain. The work gets
dealt out to the right model tier, run in parallel, and nothing lands until you
have verified it yourself.

Born from a real session that shipped in one sitting: a whole-repo
over-engineering audit (−5,600 lines, −23 dependencies), a production security
fix, two game features, a test suite for the most fragile hook, 31 lint-warning
judgment calls, and a full cosmetics economy — by running the right work on the
right tier, at the same time, with independent verification before every commit.

## The idea in one picture

```
        ┌──────────────────────────────────────┐
        │  THE GOOD  ·  top-tier model          │  plans · prioritizes
        │  your main session — the orchestrator │  integrates · verifies
        └───────────────────┬──────────────────┘   owns every risky call
                            │  fenced task prompts, dealt in parallel
              ┌─────────────┴─────────────┐
              ▼                           ▼
   ┌────────────────────┐      ┌────────────────────┐
   │  THE BAD           │      │  THE UGLY          │   each runs in its own
   │  strong model      │      │  fast model        │   herdr pane — a live
   │  work needing      │      │  well-specified    │   gun you watch fire
   │  taste & judgment  │      │  grunt work        │
   └────────────────────┘      └────────────────────┘
```

## The three layers

The layers *are* model tiers. You place work on a layer by asking one question:
**how much does the prompt have to decide?**

| Layer | Who runs it | What it gets | The test |
|-------|-------------|--------------|----------|
| **The Good** | You — the top model in the session | Planning, prioritization, integration, final verification, anything security- or money-adjacent | *Decides **what the goal is***. Never delegated. |
| **The Bad** | A strong model (e.g. Opus) | Refactors needing taste, test suites for gnarly code, case-by-case fixes (each lint warning judged on its merits) — the ruthless work | *Decides **whether** as well as **how***. |
| **The Ugly** | A fast model (e.g. Sonnet) | Migrations, renames, dedups, mechanical wiring — anything you can fully specify up front | *Only decides **how**; the outcome is already pinned.* |

> If a prompt can fully specify the outcome, it is Ugly work — hand it to the
> fast model. If the agent must exercise judgment, it is Bad work — spend the
> strong model. If it has to decide the objective itself, it stays with The
> Good. That's you.

Spending Opus on a mechanical rename is as wrong as handing a judgment refactor
to a fast model. Tier matching is the whole game.

## How orchestration works

The Good runs a five-step loop. Scale it to the job — a one-file fix needs zero
agents; only a genuine multi-front session earns the full ceremony.

1. **Scope with an audit, not a guess.** Before delegating anything, produce a
   *ranked findings list* yourself — read the tree, measure, count usages. Rank
   by value-per-effort, biggest cut first. Pure deletions rank above refactors:
   they're CI-verifiable and carry zero design risk. Trivial items you do inline;
   only real work becomes an agent task. (`/gbu-audit` does exactly this phase.)

2. **Fan out with fenced scopes.** Deal independent agents *in one move* so they
   run at once. Every task prompt carries five things — see
   [the fenced-prompt template](docs/orchestration.md#the-fenced-prompt). The
   iron rule: **no two concurrent agents may write the same file.** Every prompt
   names the files it must *not* touch and who owns them instead.

3. **Herd the guns.** An agent goes quiet without a report → message it and
   demand one; silence is not success. A spawn fails → retry once, then do it
   inline rather than block. An agent blames a failure on "someone else" →
   check whether *you* caused it with a concurrent edit. (`/gbu-status`.)

4. **Verify independently, then commit.** An agent's "all green" is a *claim*.
   Re-run the full typecheck + test suite yourself before committing its work as
   its own commit, with a message saying what changed *and why*.

5. **Close the loop.** Restate the findings that matter to the user (they never
   saw the agent's report). Log the session before context runs out. Fix stale
   docs the moment you notice they lie.

The full field guide — the prompt template, the verification protocol, the
spec→plan→execute gate for features, and the pitfalls list — lives in
[docs/orchestration.md](docs/orchestration.md).

## herdr: the situation room

There are two ways to fan out, and the herdr one is what makes this a *framework*
you can watch instead of a black box.

**In-session sub-agents.** Ask The Good to orchestrate and it spawns headless
sub-agents that run concurrently and report back. Zero setup, right for most
tasks. You see summaries, not keystrokes.

**The herdr situation room.** When you want to *watch* the executors work — and,
critically, pin a different model tier to each — give every gunslinger its own
pane running a separate Claude process. herdr is an agent multiplexer built for
exactly this: named agents, per-pane state, `agent wait` on completion. The tier
binding is one flag: `claude --model`.

```bash
# The Good stays in your live herdr pane and writes each executor a fenced brief
# (see docs/orchestration.md) into .gbu/. Then deal the panes — each spawns as a
# named agent split off your pane, tier bound via --model, brief handed over:

herdr agent start bad  --split right -- claude --model opus   "$(cat .gbu/bad.md)"   # right pane → The Bad
herdr agent start ugly --split down  -- claude --model sonnet "$(cat .gbu/ugly.md)"  # below      → The Ugly

# No attach step — they appear in your UI. Watch, then collect:
herdr agent read ugly --source recent --lines 40   # peek at output
herdr agent wait ugly --status idle                # block until it finishes
```

Now the metaphor is literal: three panes, three tiers, three jobs running side
by side. You keep your pane (The Good) to integrate and verify; the executors
work in theirs. Use interactive `claude "…"` when you want to interject
mid-draft, or `claude -p "…"` for fire-and-forget headless runs whose output you
collect at the end. Either way, **the fences in each brief are what keep two
panes from writing the same file** — that discipline matters more here than
anywhere, because now the collisions are real processes, not one model's
sequential edits.

## Install

```
/plugin marketplace add XyndoX/good-bad-ugly
/plugin install good-bad-ugly@good-bad-ugly
```

…or from a local clone:

```bash
claude plugin install /path/to/good-bad-ugly
```

Then just ask Claude to "orchestrate" / "conduct" a big multi-part task, or:

```
/gbu tackle the ranked findings from this audit
```

## Commands

| Command | What it does |
|---------|--------------|
| `/gbu <task>` | Run the full orchestration loop on a large, multi-part task. |
| `/gbu-audit` | The scope-and-rank phase only — a ranked findings list, no delegation, no edits. |
| `/gbu-status` | Herd the running agents: who's idle, who owes a report, whose work is unverified, who's colliding. |
| `/gbu-retro` | Post-session retrospective — harvest the session's scars into durable doctrine (a rule, a pitfall, a fence) instead of a summary nobody rereads. |

## What's inside

- **`skills/good-bad-ugly/SKILL.md`** — the workflow The Good runs: the tier
  ladder, the fan-out loop, agent herding, external-system discipline, the
  spec→plan→execute gate, and a field-tested pitfall list.
- **`commands/`** — the four slash commands above.
- **`agents/`** — `the-bad` (strong model) and `the-ugly` (fast model) as
  typed, spawnable executors with the tier pre-bound and the iron rules
  (fences, disposable fixtures, paid-API caution, honest reports) baked into
  their definitions — tier matching enforced by structure, not memory.
- **`docs/orchestration.md`** — the operational playbook: the fenced-prompt
  template, the herdr recipes, the verification protocol.
- **`AGENTS.md`** — the condensed workflow for non-Claude harnesses.

## Pairs well with

- [superpowers](https://github.com/obra/superpowers) — the
  brainstorm→spec→plan→execute pipeline The Good uses for feature work.
- A minimalism skill like **ponytail** — good-bad-ugly decides *who* does the
  work; a minimalism skill decides *how much* work there should be. Run the
  audit through both and you cut the right things with the right hands.

## FAQ

**Do I have to use herdr?**
No. In-session sub-agents need zero setup and cover most work. herdr is for when
you want to watch executors live and pin a distinct model tier to each pane.

**Isn't spawning agents just overhead?**
Yes — overhead you must earn back in parallelism. One workstream? Do it inline.
The loop only pays off when independent work can genuinely run at the same time.

**Why the western names?**
The Good plans and shoots straight. The Bad does the work that takes nerve. The
Ugly does the digging nobody wants. Same movie, same division of labor.

## License

[MIT](LICENSE).

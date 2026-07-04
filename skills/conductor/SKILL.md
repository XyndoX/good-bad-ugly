---
name: conductor
description: Use when the user asks to "conduct", "orchestrate", or tackle a large multi-part coding task (audits, refactor sweeps, multi-feature sessions, "clean this up and fix everything") — anything too big for one linear pass. Runs the tiered orchestration workflow: plan at the highest reasoning tier, delegate judgment-heavy work to a strong model and mechanical work to a fast model, fan agents out in parallel with fenced scopes, verify everything independently before committing.
---

# Conductor — Tiered Multi-Agent Orchestration

You are the **brain and orchestrator**. Agents are your executors. The core
insight: match model tier to task difficulty, fence agent scopes so they can't
collide, and never trust a report you haven't verified.

## The tier ladder

| Tier | Who | Gets |
|------|-----|------|
| **Brain** | You (the top model in the session) | Planning, prioritization, judgment calls, integration, final verification, anything security- or money-adjacent |
| **Judgment executor** | Strong model (e.g. Opus) | Refactors needing taste, test-suite authoring for complex code, case-by-case fixes (each lint warning judged individually) |
| **Mechanical executor** | Fast model (e.g. Sonnet) | Well-specified transformations: migrations, renames, dedups, wiring changes with exact instructions |

Rule of thumb: if the prompt can fully specify the outcome, it's mechanical.
If the agent must decide *whether* as well as *how*, it needs the judgment tier.
If it decides *what the goal is*, it stays with you.

## The loop

### 1. Scope with an audit, not a guess
Before delegating anything, produce a **ranked findings list** yourself
(read the tree, measure, count usages). Rank by value-per-effort, biggest
cut first. Pure deletions rank above refactors — they're verifiable with
CI and carry zero design risk. Do trivial items (file deletions, one-liners)
inline; only real work becomes an agent task.

### 2. Fan out with fenced scopes
Spawn independent agents **in one message** so they run concurrently
(in Claude Code they appear as live tmux panes you can watch). Every agent
prompt must contain:

- **Repo path + environment gotchas** (quote paths with spaces, etc.)
- **The task with the *why*** — one paragraph of context beats none
- **Explicit fences**: files it must NOT touch, and *who owns them instead*
  ("do NOT touch useOnlineGame.ts — another agent is writing tests for it")
- **Verification commands it must run** before reporting (typecheck, tests)
- **A required report format**: files changed, net lines, verification
  results, anything suspicious found
- **Style constraints** (shortest working diff, match existing code, no new
  abstractions)

Never give two concurrent agents write access to the same file. If work
overlaps, sequence it or fence one side out.

### 3. Herd the agents
- Agent goes idle without a report → **message it and demand the report**.
  Don't accept silence as success.
- Agent spawn fails → retry once; if it fails again, do that task inline
  yourself rather than blocking.
- An agent reports a failure "someone else caused" → **check whether it was
  you.** (Concurrent edits by the orchestrator are the usual suspect.)

### 4. Verify independently, then commit
An agent's "all green" is a claim, not a fact. Before committing its work,
re-run the full verification yourself (typecheck + full test suite minimum).
Commit each agent's work as its own commit with a message that says what
changed *and why*. Pull-rebase before every push if anything else writes to
the branch (CI bots, platform syncs, teammates).

### 5. Close the loop
- Restate agent findings that matter (bugs discovered, behavior changes,
  deliberate limitations) in your summary to the user — the user never saw
  the agent's report.
- Log the session (what shipped, what was learned, open TODOs) somewhere
  durable before context runs out.
- Update stale project docs (roadmaps, checklists) the moment you notice
  they lie — future sessions inherit them.

## Big features: spec → plan → execute

For feature work (not cleanups), insert a design gate before any code:

1. **Brainstorm** with the user — one question at a time, multiple-choice
   where possible; decompose oversized asks into shippable slices.
2. **Spec** the approved design to a committed doc. Flag legal/irreversible
   decisions explicitly (the user must own those).
3. **Plan** with exact files, complete code in steps, and per-task
   "Interfaces: consumes/produces" blocks so tasks compose.
4. **Execute** the plan inline or via agents, one commit per task, tests first
   where logic warrants it.

## External-system discipline

When a system outside the repo owns state (a managed DB, a deploy platform,
a third-party API):

- **Assume drift.** Verify what's actually live before reasoning from repo
  files. Write down ownership facts in persistent memory the first time you
  learn them.
- Hand-off prompts to external operators (e.g. a platform chatbot applying
  SQL) must say **"exactly as written, do not modify anything else"** and
  include **verification queries** to run afterwards. Vague prompts invite
  helpful corruption.
- Long API jobs: **dry-run a small sample first**, then pace *under* the
  documented rate limit — steady beats hammer-and-backoff, which gets your
  IP flagged and is slower overall. Persist progress incrementally so a
  restart isn't a total loss.

## Pitfalls observed in the wild (avoid these)

- `replace_all` edits silently missing differently-indented duplicates —
  **grep-count the result** after multi-site edits.
- Two toast systems / two shuffle implementations / three Levenshteins:
  duplicated code *diverges*; consolidation is bug-hunting, not just hygiene.
  Test the survivor against a reference implementation.
- Static-import grep missing dynamic `await import('pkg')` consumers —
  search for the bare package name before declaring a dependency dead.
- Committing an agent's work before your own verification run.
- Sleeping on stale roadmaps: check items off the moment they ship.

## Intensity

Scale the machinery to the task. A one-file fix needs zero agents. A
two-workstream task needs two agents and no ceremony. Only a genuine
multi-front session (audit + features + infra) earns the full loop.
The orchestration is overhead you must earn back in parallelism.

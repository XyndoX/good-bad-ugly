---
name: good-bad-ugly
description: Use when the user asks to "orchestrate", "conduct", invoke "good bad ugly" / "gbu", or tackle a large multi-part coding task (audits, refactor sweeps, multi-feature sessions, "clean this up and fix everything") — anything too big for one linear pass. Runs the tiered orchestration workflow: The Good (brain) plans and verifies, The Bad (judgment executor, strong model) handles work needing taste, The Ugly (mechanical executor, fast model) does the grunt work — fanned out in parallel with fenced scopes, everything independently verified before committing.
---

# The Good, the Bad and the Ugly — Tiered Multi-Agent Orchestration

Three gunslingers, three model tiers. You are **The Good** — the brain and
orchestrator. Agents are your executors. The core insight: match model tier
to task difficulty, fence agent scopes so they can't collide, and never trust
a report you haven't verified.

## The tier ladder

| Gunslinger | Who | Gets |
|------|-----|------|
| **The Good** (brain) | You (the top model in the session) | Planning, prioritization, judgment calls, integration, final verification, anything security- or money-adjacent |
| **The Bad** (judgment executor) | Strong model (e.g. Opus) | Refactors needing taste, test-suite authoring for complex code, case-by-case fixes (each lint warning judged individually) — the ruthless work |
| **The Ugly** (mechanical executor) | Fast model (e.g. Sonnet) | Well-specified transformations: migrations, renames, dedups, wiring changes with exact instructions — the dirty work |

Rule of thumb: if the prompt can fully specify the outcome, it's work for
The Ugly. If the agent must decide *whether* as well as *how*, it needs
The Bad. If it decides *what the goal is*, it stays with The Good — you.

## The loop

### 1. Scope with an audit, not a guess
Before delegating anything, produce a **ranked findings list** yourself
(read the tree, measure, count usages). Rank by value-per-effort, biggest
cut first. Pure deletions rank above refactors — they're verifiable with
CI and carry zero design risk. Do trivial items (file deletions, one-liners)
inline; only real work becomes an agent task.

### 2. Fan out with fenced scopes
Spawn independent agents **in one message** so they run concurrently. Two modes:

- **In-session sub-agents** (default) — headless, zero setup, they report back.
  This plugin ships the executors as typed agents with the tier pre-bound:
  spawn `the-bad` (strong model) for judgment work and `the-ugly` (fast model)
  for mechanical work — tier matching enforced by the agent definition, not by
  remembering a flag.
- **tmux situation room** — one pane per executor, each a separate `claude`
  process, with the model tier bound per pane: `claude --model opus …` for
  The Bad, `claude --model sonnet …` for The Ugly. Use when you want to watch
  the executors work live or pin distinct tiers. Recipe + prompt template:
  `docs/orchestration.md`.

Every agent prompt must contain:

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
- **Messages cross.** Async agents compose replies before reading your latest
  directive. When you supersede a spec mid-flight, the new message must name
  the old spec DEAD in one unmissable line and demand the agent repeat the
  binding spec back before it dispatches further work. Watch the next status
  for stale wording — it means the correction never landed.

### 4. Verify independently, then commit
An agent's "all green" is a claim, not a fact. Before committing its work,
re-run the full verification yourself (typecheck + full test suite minimum,
plus the project's **domain gates** — benchmarks, golden hashes, visual
checks — not just the generic suite; an executor's green tests can coexist
with a wrecked benchmark). Two verification traps that pass silently:

- **Verify the path you're shipping.** Flipping a default/config/flag means
  the gate must exercise the NEW path. A benchmark that still runs the old
  path is vacuously green. (Real case: a default-ON flip whose bench had only
  ever measured the OFF path — the ON path hid a 74× force regression.)
- **Read the artifact.** When the deliverable is a generated artifact (PDF,
  image, bundle, export), open and inspect the artifact itself — and check
  its SIZE. (Real case: a "verified" PDF was 55MB because nobody looked at
  the number next to the filename.)

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

## Data & spend safety (iron rules for every brief)

Two rules that must appear in any brief whose work touches live-ish systems.
Both were learned the expensive way:

- **Executors never mutate real data.** Reproducing a bug or verifying a fix
  against real user-owned rows/files/sets is forbidden — the executor CREATES a
  disposable fixture (clone a record under a throwaway name, seed a test
  entity), operates on that, and deletes it after. A brief that says
  "reproduce first" without naming a disposable fixture is a loaded gun.
  (Real case: a diagnosis phase re-ran a destructive operation across 12 real
  production records; the overwrites were irreversible.)
- **Every external call is PAID until proven free.** Provider selection often
  hides in env vars — the "mock" provider you assume is active may be the
  real, billed API. Verify which provider the environment resolves before any
  agent triggers generation/inference/sending. (Real case: "free mock" QA
  burned real API credits because a key in .env.local silently selected the
  paid provider.)

Corollary: a **destructive user-facing operation** (overwrite-in-place,
regenerate, migrate) should get its undo/backup designed BEFORE agents are
allowed to exercise it. If one click can destroy state, an agent will
eventually click it.

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
- Unanchored patterns in ignore files (`.vercelignore`, `.dockerignore`)
  matching nested dirs — a bare `supabase` also excludes `lib/supabase/` and
  breaks the build. Anchor to root (`/supabase`).
- Retrying a stuck upload/deploy without killing the previous attempt —
  orphaned processes compete for bandwidth and every retry gets slower.
  `pkill` first, then retry once, clean.
- Test timeouts tuned on your machine flaking on CI's 2-core runners — a
  timeout is a hang guard, not a perf gate; size it generously (30s local
  work can take 35s+ on CI).
- Users watch the hot-reloading dev server while executors edit — mid-flight
  WIP looks like shipped garbage. Warn the user, or fix the visible page
  first.

## Intensity

Scale the machinery to the task. A one-file fix needs zero agents. A
two-workstream task needs two agents and no ceremony. Only a genuine
multi-front session (audit + features + infra) earns the full loop.
The orchestration is overhead you must earn back in parallelism.

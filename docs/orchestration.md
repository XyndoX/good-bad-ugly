# Orchestration field guide

The operational detail behind the [README](../README.md). If the README is the
map, this is the manual for actually running a session.

- [Choosing the tier](#choosing-the-tier)
- [The fenced prompt](#the-fenced-prompt)
- [tmux recipes](#tmux-recipes)
- [The verification protocol](#the-verification-protocol)
- [Big features: spec → plan → execute](#big-features-spec--plan--execute)
- [External-system discipline](#external-system-discipline)
- [Pitfalls seen in the wild](#pitfalls-seen-in-the-wild)

---

## Choosing the tier

Decide the tier by how much the *prompt* has to decide. Walk it top-down and
stop at the first "yes":

```
Does the task decide WHAT the goal is?          → The Good  (you, keep it)
Does it decide WHETHER as well as HOW?          → The Bad   (strong model)
Is the outcome fully specifiable up front?      → The Ugly  (fast model)
```

Concrete calls from a real session:

| Task | Tier | Why |
|------|------|-----|
| Rank the audit findings by value-per-effort | The Good | Decides the objective and the order of battle. |
| Judge 31 lint warnings one by one, fix or suppress each | The Bad | Every warning is a separate *whether*, not just a *how*. |
| Author a test suite for the most fragile hook | The Bad | Requires taste about what's worth asserting. |
| Rename a symbol across 40 files; dedupe two shuffle impls | The Ugly | Outcome is pinned; only the mechanics remain. |
| Delete a dead speculative feature and its imports | The Ugly | Pure deletion, CI-verifiable, zero design risk. |
| Decide whether the cosmetics economy ships at all | The Good | That's a product call — it never leaves you. |

Miscasting is expensive both ways: a fast model fumbles judgment work, and a
strong model burns time and tokens on a mechanical rename. Match the tier.

## The fenced prompt

Every task you hand an executor — sub-agent or tmux pane — carries **five**
things. Drop any of them and you get drift, collisions, or a confident "done"
that isn't.

```md
## Task: <one-line goal>

**Repo:** /abs/path/to/repo   (quote paths with spaces; note env gotchas —
  e.g. use `gtimeout` not `timeout`; zsh needs quoted "[bracket]" globs)

**Why:** <one paragraph of context>. A paragraph of intent beats a page of
  instructions — it lets the agent make the small calls correctly.

**Do this:** <the actual work, specified to the tier — exact steps for The
  Ugly, goal + constraints for The Bad>.

**Fences — do NOT touch:**
  - src/useOnlineGame.ts  ← The Bad owns this (writing its tests)
  - any file under migrations/  ← sequenced after you, not concurrent
  (You may ONLY write: <explicit allowlist>.)

**Verify before reporting:** run `<typecheck cmd>` and `<test cmd>`; both must
  pass. Do not report green on an unrun check.

**Report back:** files changed · net lines ±  · verification results ·
  anything suspicious you noticed but didn't fix.
```

The fence is the load-bearing part. **No two concurrent executors may write the
same file.** If work overlaps, either sequence it (one after the other) or fence
one side out and give that slice to a single owner. Name the owner in the prompt
so the agent knows a locked file isn't a bug — it's someone else's gun.

## tmux recipes

The situation room: one pane per gunslinger, the model tier bound per pane with
`claude --model`.

### Deal a three-pane showdown

```bash
# The Good (you) writes each brief into .gbu/ first, then:
tmux new-session -d -s gbu -n showdown
tmux split-window -h  -t gbu:showdown            # right        → The Bad
tmux split-window -v  -t gbu:showdown.1           # bottom-right → The Ugly
tmux select-layout -t gbu:showdown tiled

tmux send-keys -t gbu:showdown.1 'claude --model opus   "$(cat .gbu/bad.md)"'  Enter
tmux send-keys -t gbu:showdown.2 'claude --model sonnet "$(cat .gbu/ugly.md)"' Enter

tmux attach -t gbu:showdown
```

### Interactive vs. headless

- `claude --model … "$(cat brief.md)"` — **interactive**. The pane stays live;
  you can interject, answer a question, or redirect mid-draft. Use when the work
  is exploratory or you want a hand on the wheel.
- `claude -p --model … "$(cat brief.md)"` — **headless print mode**. Runs to
  completion, prints the result, exits. Use for fire-and-forget mechanical work
  whose output you collect at the end. Redirect it: `… -p … > .gbu/ugly.out`.

### Watch, then collect

```bash
tmux capture-pane -t gbu:showdown.2 -p        # snapshot a pane's output
tmux kill-session -t gbu                       # strike the set when done
```

When every pane reports done, control returns to The Good: you re-verify each
executor's work (next section) and commit it yourself, one commit per gun.

## The verification protocol

An executor's report is a claim, not a fact. Before **anything** an agent
touched enters a commit:

1. **Re-run the full gate yourself** — the project's typecheck *and* full test
   suite, not the subset the agent ran. Green on the agent's machine is not
   green on yours.
2. **Grep-count multi-site edits.** After a `replace_all` or a cross-file
   rename, count the occurrences that should remain and confirm the number.
   Silent misses on differently-indented duplicates are the classic failure.
3. **Diff for scope creep.** Confirm the agent only wrote inside its allowlist.
   A file changed outside the fence is a collision or a misunderstanding — catch
   it before it compounds.
4. **Commit per gun.** One executor's work = one commit, message says what
   changed *and why*. Pull-rebase before every push if anything else writes the
   branch (CI bots, platform syncs, teammates).

Only after your own gate is green does the work exist. Committing on an agent's
say-so is how a bad edit reaches `main`.

## Big features: spec → plan → execute

Cleanups can go straight to fan-out. *Features* get a design gate first — code
before design is how scope explodes.

1. **Brainstorm** with the user: one question at a time, multiple-choice where
   you can. Decompose an oversized ask into shippable slices.
2. **Spec** the approved design into a committed doc. Flag every legal or
   irreversible decision explicitly — the user must own those, not you.
3. **Plan** with exact files, complete code per step, and a per-task
   *Interfaces: consumes / produces* block so tasks compose without a meeting.
4. **Execute** the plan — inline or fanned out, one commit per task, tests first
   where the logic warrants it.

## External-system discipline

When state lives outside the repo — a managed DB, a deploy platform, a
third-party API — the repo files are a hypothesis, not the truth.

- **Assume drift.** Verify what's actually live before reasoning from repo files.
  Write ownership facts into persistent memory the first time you learn them.
- **Hand-off prompts to external operators** (a platform chatbot applying SQL,
  say) must end with *"exactly as written, do not modify anything else"* and
  include the **verification queries** to run afterwards. A vague hand-off
  invites helpful corruption.
- **Long API jobs:** dry-run a small sample first, then pace *under* the
  documented rate limit. Steady beats hammer-and-backoff — the latter gets your
  IP flagged and is slower overall. Persist progress incrementally so a restart
  isn't a total loss.

## Pitfalls seen in the wild

- **`replace_all` skipping differently-indented duplicates.** Grep-count the
  result after any multi-site edit.
- **Duplicated code diverges.** Two toast systems, two shuffles, three
  Levenshteins — consolidation is bug-hunting, not hygiene. Test the survivor
  against a reference implementation.
- **Dynamic-import blind spots.** A static-import grep misses
  `await import('pkg')`. Search the bare package name before declaring a
  dependency dead.
- **Committing before your own verification run.** The whole point of the
  protocol above.
- **Sleeping on stale roadmaps.** Check items off the moment they ship; the next
  session inherits whatever the docs say.
- **Blaming a phantom.** When an agent reports a failure "someone else caused,"
  the usual suspect is your own concurrent edit. Check yourself first.

---
description: Run the tiered multi-agent orchestration loop on a large task
argument-hint: [what to tackle]
---

Orchestrate this task using the good-bad-ugly workflow (`skills/good-bad-ugly/SKILL.md`): $ARGUMENTS

You are **The Good** — the brain. Run the loop:

1. **Scope with an audit, not a guess** — produce a ranked findings list yourself before delegating anything. Do trivial items inline; only real work becomes an agent task.
2. **Fan out with fenced scopes** — spawn independent agents in one message. Match tier to task: fully-specifiable work → **The Ugly** (fast model); work needing judgment → **The Bad** (strong model). Every agent prompt gets: repo path + gotchas, the task with the *why*, explicit file fences (what NOT to touch + who owns it), verification commands, a required report format. Never give two concurrent agents write access to the same file.
3. **Herd** — demand reports from silent agents; retry a failed spawn once then do it inline; when an agent blames "someone else," check whether it was your own concurrent edit.
4. **Verify independently, then commit** — re-run typecheck + full tests yourself before committing each agent's work as its own commit.
5. **Close the loop** — restate agent findings that matter to the user, log the session, update stale docs.

Scale the machinery to the task — a one-file fix needs zero agents.

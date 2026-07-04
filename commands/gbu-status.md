---
description: Herd the running agents — who's idle, who owes a report, who to verify
---

Herd the currently running good-bad-ugly agents. For each one report:

- **State** — working, idle, done, or failed.
- **Owes a report?** — if it went idle without reporting, message it and demand the report format (files changed, net lines, verification results, anything suspicious). Don't accept silence as success.
- **Verified?** — an agent's "all green" is a claim, not a fact. Flag any work not yet independently re-verified by you (typecheck + full tests) before it gets committed.
- **Scope conflicts** — any two agents touching the same file, or an agent blaming a failure on "someone else" that was actually your own concurrent edit.

Finish with the next action: who to poll, whose work to verify-and-commit, what to re-spawn or fold inline.

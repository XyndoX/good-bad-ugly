---
description: Scope-and-rank phase only — a ranked findings list, no delegation
---

Run only the **scope** phase of the good-bad-ugly workflow — no agents, no delegation, no edits.

Read the tree, measure, count usages. Produce a **ranked findings list**, biggest value-per-effort first. Pure deletions rank above refactors — they're CI-verifiable and carry zero design risk. For each finding give one line: what to do, why it's worth it, rough effort, and which tier would own it (**The Good** = you keep it; **The Bad** = judgment executor; **The Ugly** = mechanical executor).

End with the shape of a delegation plan: which findings could run as concurrent agents, and where their file scopes would need fencing so no two collide. Stop there — don't execute.

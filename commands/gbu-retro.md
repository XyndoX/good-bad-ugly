---
description: Post-session retrospective — harvest this session's lessons into durable doctrine
---

Run a good-bad-ugly retrospective on the current session. The goal is not a
summary — it is to harvest anything that should change how future sessions
run, and put it where future sessions will actually see it.

1. **Sweep the session for scars.** Incidents (data touched that shouldn't
   have been, money spent unexpectedly, a gate that passed while the product
   was broken), corrections the user had to make twice, specs that got
   superseded mid-flight, agents that went quiet, verification that turned
   out vacuous.

2. **For each scar, ask: which layer failed?** A brief missing a fence or a
   fixture rule → the prompt template needs the rule. A verification gap →
   the verify step needs the check. A user-facing surprise → the loop needs a
   communication step. If it couldn't have been prevented by better doctrine,
   say so — not everything is a process failure.

3. **Write the durable artifact, smallest first:**
   - a one-line addition to an existing pitfalls/rules list beats a new section;
   - a new section beats a new file;
   - a new file needs to earn its keep.
   Update the project's own docs/memory for project-specific lessons; update
   this plugin's SKILL.md/docs for workflow-general ones. Real cases beat
   abstract rules — cite what actually happened, anonymized to its mechanism.

4. **Report to the user**: what was harvested, where it landed, and the one
   lesson most worth remembering. Skip ceremony if the session was clean —
   "no doctrine-worthy scars" is a valid outcome.

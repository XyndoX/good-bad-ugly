# 🎼 claude-conductor

A [Claude Code](https://claude.com/claude-code) plugin that turns Claude into a
**tiered multi-agent orchestrator** for large coding sessions.

Born from a real session that shipped, in one sitting: a whole-repo
over-engineering audit (−5,600 lines, −23 dependencies), a production security
fix, two game features, a test suite for the most fragile hook, 31 lint-warning
judgment calls, and a full cosmetics economy — by running the right work on the
right model tier, in parallel, with independent verification.

## The idea

```
        ┌─────────────────────────────┐
        │  BRAIN (top-tier model)     │  plans, prioritizes, integrates,
        │  = your main session        │  verifies, owns risky decisions
        └──────────┬──────────────────┘
                   │ fenced task prompts, spawned in parallel
        ┌──────────┴──────────┐
        ▼                     ▼
┌───────────────────┐  ┌───────────────────┐
│ JUDGMENT executor │  │ MECHANICAL executor│   run as live tmux panes
│ (strong model)    │  │ (fast model)       │   you can watch working
│ refactors, tests, │  │ migrations, dedups,│
│ case-by-case fixes│  │ well-specified work│
└───────────────────┘  └───────────────────┘
```

Three rules make it work:

1. **Tier matching** — if a prompt can fully specify the outcome, it's
   mechanical (fast model). If the agent decides *whether* as well as *how*,
   it needs judgment (strong model). If it decides *what the goal is*, it
   stays with the brain.
2. **Scope fencing** — no two concurrent agents may write the same file.
   Every agent prompt names the files it must NOT touch and who owns them.
3. **Independent verification** — an agent's "all green" is a claim.
   The brain re-runs typecheck + tests before committing anything.

## Install

```bash
# from a local clone
claude plugin install /path/to/claude-conductor

# or add as a marketplace source in Claude Code settings
```

Then in any session:

```
/conductor tackle the ranked findings from this audit
```

…or just ask Claude to "orchestrate" / "conduct" a big multi-part task —
the skill triggers on that intent.

## What's inside

- `skills/conductor/SKILL.md` — the full workflow: the tier ladder, the
  fan-out loop, agent herding (demanding reports from silent agents),
  external-system discipline (managed DBs, rate-limited APIs), the
  spec → plan → execute gate for feature work, and a field-tested pitfall
  list (divergent duplicated code, dynamic-import blind spots,
  `replace_all` indentation misses, and more).

## Pairs well with

- [superpowers](https://github.com/claude-plugins/superpowers) — the
  brainstorm → spec → plan → execute pipeline conductor uses for features
- A minimalism skill (ponytail-style) — conductor decides *who* does the
  work; a minimalism skill decides *how much* work there should be

## License

MIT

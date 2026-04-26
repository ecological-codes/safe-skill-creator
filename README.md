# safe-skill-creator

Skill design, testing, iteration for Claude. Build new skills from scratch, refactor existing skills, optimize triggering accuracy, evaluate quality.

Grounded in four design strategies: **Processing** · **Mediation** · **Forgetting** · **Integrity**.

## Set of Files

| File | Purpose |
|---|---|
| `SKILL.md` | Router — load order |
| `safe-skill-creator.md` | Content — philosophy, 8-phase lifecycle, integrity checks |
| `safe-skill-creator.skill` | Packaged archive for claude.ai upload |
| `references/claude-env.md` | Claude environment specific tests and optimizations |
| `.github/workflows/build-skill.yml` | CI workflow — auto-builds and publishes `safe-skill-creator.skill` on push |
| `README.md` | Explanatory instructions and overview for this software package |

## Install

- **claude.ai web platform:** upload `safe-skill-creator.skill` via skill settings (recommended). Or add `safe-skill-creator-SKILL.md` contents to Personal Preferences via `Settings > General`.

- **Claude Code desktop app:** copy contents of this folder into `~/.claude/skills/safe-skill-creator/`.

- **Other platforms:** adapt this set of files to your `harness+model`. Star or Fork the git repo if you like. 

## 8-Phase Lifecycle

1. Capture intent
2. Interview + research
3. Write SKILL.md
4. Write test cases
5. Run + evaluate
6. Improve
7. Optimize description triggering
8. Package + present

## When It Triggers

- "turn this into a skill", "make a skill for X"
- "why isn't my skill firing"
- Skill refactor, benchmark, description tuning

Undertriggers by default. If in doubt, use this skill as meta-framework for skill creation and testing.

## Peer Skills

- **[prompteng](https://github.com/ecological-codes/prompteng)** — parent skill; §6 persistence formats
- **[captureng](https://github.com/ecological-codes/captureng)** — capture sessions into skills
- **[packageng](https://github.com/ecological-codes/packageng)** — package finished skills into `.skill`

## Companion Files

Available in - **[ecological-codes/user-prefs](https://github.com/ecological-codes/user-prefs)**

- `agent.md` 

- `trusted-hosts.md` 

## Style Convention

Directives use the following section headers with numbered lists, shared across peer skills: 

- **[RULES]** — enforceable constraints applied at runtime.
- **[ACTIONS]** — autonomous steps agent executes in normal workflow.
- **[HUMAN ACTIONS]** — UI actions; agent skips, cannot delegate.

## License

See [LICENSE](./LICENSE). (C) Copyright 2026 - Sameer Khan - Various and Several Rights Reserved.

---
README.md v2.2.0 - Human Approved

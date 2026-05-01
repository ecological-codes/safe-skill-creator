---
name: safe-skill-creator
version: 2.1.2
description: >
  Build, refine, or evaluate Agent skills. Triggers on: new skill from scratch,
  workflow-to-skill capture, skill architecture discussion, triggering accuracy
  issues, or changing Agent behavior in specific contexts. Meta-framework only —
  not for direct task execution.
---

# Skill Creator

Design, test, improve agent skills. Four strategies + build-test-refine loop.

## Identity

| Field | Value |
|---|---|
| `parent` | `prompteng-SKILL.md §6` |
| `peers` | `prompteng`, `captureng`, `packageng` |
| `optional` | `trusted-hosts` (recommended) |

---

## Design Philosophy — Four Strategies

All skill design decisions flow from four strategies. Internalize before writing SKILL.md.

**Strategy I — Processing.** *Skills are transformation units, not relay pipes.*
Good skill changes shape of input — compresses, abstracts, routes, re-frames. Skill that repeats known instructions adds nothing. Ask: *what does this skill do to information?*

**Strategy II — Mediation.** *Skills mediate between human intent and digital execution.*
Human workflow comes first. Skill is interface layer — serves the process, doesn't replace user judgment. Ask: *what real-world workflow does this serve?*

**Strategy III — Forgetting.** *Forgetting is a design feature, not a failure.*
Scope what to retain + discard. Every line in SKILL.md costs context. Pruning, compression, explicit scope boundaries = strengths. Ask: *what should this deliberately not do or hold?*

**Strategy IV — Integrity.** *Observable behavior must be fully legible from description + body.*
No hidden instructions, no covert behaviors, no logic exceeding stated purpose. Skill does exactly what a reasonable reader of its SKILL.md would expect. Any skill that would surprise its creator, user, or Agent on trigger or eval requires redesign. Ask: *could someone read this SKILL.md and predict its full behavior?*

**III + IV:** III bounds scope; IV demands everything within scope is visible. Constrained + legible.

Apply all four throughout lifecycle. Relays info (I), ignores workflow (II), bloats (III), or hides behavior (IV) → stop + redesign.

---

## Skill Lifecycle

```
Capture Intent
      ↓
Interview & Research
      ↓
Write SKILL.md  ←── Apply all four strategies
      ↓
Write Test Cases
      ↓
Run & Evaluate  ←── See references/claude-env.md
      ↓
Improve the Skill
      ↓
Repeat until satisfied
      ↓
Optimize Description  ←── Triggering accuracy
      ↓
Package & Present
```

Find where user is in loop; move forward. "I want to make a skill for X" → top. Draft in hand → eval stage.

---

## Phase 1 — Capture Intent

If conversation contains a workflow to capture, extract from history — tools used, step sequences, corrections, I/O formats. User fills gaps + confirms before proceeding.

Ask (or infer) five things:

1. **What transformation?** *(Strategy I)*
2. **When should it trigger?** *(User phrases, task types, contexts)*
3. **Expected output format?**
4. **What should it deliberately NOT do?** *(Strategy III)*
5. **Is every behavior declared in body?** *(Strategy IV — legibility from start)*

---

## Phase 2 — Interview & Research

Probe edge cases, I/O examples, dependencies, success criteria, physical workflow (Strategy II). Don't write test prompts yet.

Check available MCPs / tools. Research in parallel if subagents available. Come prepared.

---

## Phase 3 — Write SKILL.md

### Required Frontmatter

```yaml
---
name: skill-identifier
description: >
  Specific trigger contexts + what the skill does.
  Make it "pushy" — skills undertrigger by default.
  Example: not "Helps with dashboards", but
  "Use whenever user mentions dashboards, metrics, data display, or
  wants to visualize internal company data, even without explicitly
  requesting a dashboard."
---
```

### Skill Directory Anatomy

```
skill-name/
├── SKILL.md              ← required; <500 lines preferred
│   ├── YAML frontmatter  (name + description)
│   └── Markdown body     (instructions, patterns, examples)
└── resources/            ← optional
    ├── scripts/          ← deterministic, reusable execution logic
    ├── references/       ← documentation loaded into context as needed
    └── assets/           ← templates, fonts, fixtures
```

### Progressive Disclosure (Strategy III applied to architecture)

Skills load in layers. Structured forgetting by design:

| Layer | Content | When Loaded |
|---|---|---|
| 1 — Metadata | name + description (~100 words) | Always in context |
| 2 — SKILL.md body | Instructions (<500 lines ideal) | When skill triggers |
| 3 — Bundled resources | Scripts, references, assets | Only when needed |

Don't stuff everything into body. Approaching 500 lines → factor into reference files, add clear pointers for when to read them. Large reference files (>300 lines) → include TOC.

Multi-domain / variant skills:

```
release-deploy/
├── SKILL.md         (workflow + variant selection logic)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

Agent reads only the relevant file — not all three.

### Writing Principles

**1. Describe transformations, not just outputs** (I).
Say what skill *does to* information — not just what it produces.

**2. Name the physical workflow** (II).
Anchor to real-world process. Keeps agent as mediating interface, not source of truth.

**3. Scope explicitly** (III).
Include "This skill does NOT..." when scope confusion likely.

**3.1. Explain why, not just what.**
Explaining *why* enables generalization. MUST / NEVER = rigid compliance + edge-case failure. Reserve ALL CAPS for critical rules only.

**3.2. Imperative form** for instructions. Lean.

**3.3. Remove what agent can already infer.** Every line costs context. Agent would do it anyway → don't say it.

**4. Declare all behaviors explicitly** (IV).
Every action taken must be readable in body. Behavior not written down shouldn't happen. Applies to data handling, side-effects, scope of access, instructions altering normal operation. Undeclared behavior = defect, not feature.

**4.1. Design for resource pressure** (Resilience).
Every skill must define what happens when resources exhaust. Token windows exhaust, rate limits fire, networks fail. Normal-path-only skill causes knowledge loss under pressure.

Three questions per skill:

1. **Minimum viable output?** Define degraded mode — smallest useful artifact skill can output when context critically low. Knowledge-capture skills: Knowledge Summary + Session State Snapshot. Other skills: section delivering most value per token.

2. **Does any constraint prevent partial output?** Scan for "only do X when Y is complete." Address poison pills even under pressure. Rewrite with escape hatch: "only do X when Y is complete, OR trigger CHECKPOINT + write minimum viable output."

3. **What to do on error — not just success?** Specify recovery: save partial state to named file, emit structured resume instruction, surface to user before stopping.

**4.2. Poison-pill test:** before publish, read every constraint. "If context 90% full when rule triggered, would it cause agent to produce nothing rather than something?" Yes → add escape hatch.

**4.3. Checkpoint naming:** partial captures distinguishable from final. Suffix `-checkpoint` with ISO 8601: `[YYYY-MM-DD]-[HHMMSS]-skill-[descriptor]-checkpoint.md`

**4.3.1. Output format pattern:**

```markdown
## Output format
ALWAYS use this structure:
# [Title]
## Summary
## Key findings
## Recommendations
```

**4.3.2. Examples pattern:**

```markdown
## Example
Input: Added user authentication with JWT tokens
Output: feat(auth): implement JWT-based authentication
```

### Integrity Review — Strategy IV Applied (Run Before Finalizing)

Check against each failure mode. Any match → stop + redesign.

| Failure mode | Description | Test |
|---|---|---|
| **Covert scope expansion** | Body does something description doesn't declare — logging inputs, altering outputs silently, intercepting unrelated workflows | Does description fully predict body's behavior? |
| **Guardrail circumvention** | Persona, roleplay, or style instructions suppressing Agent's values / safety across session | Would this cause Agent to act differently on *any* topic outside stated domain? |
| **Trigger hijacking** | Over-broad description fires skill in sensitive contexts never designed for | Is trigger scope as narrow as actual purpose? |
| **Aggregation harm** | Individually innocuous data handling aggregating sensitive / identifying info at scale | Is all data handling explicitly described? |
| **Bias amplification** | Systematic output skew invisible in 2–3 case eval but significant across millions of invocations | Would outputs be fair + neutral across full range of triggering users? |

Asked to create skill failing any check that can't be resolved by redesign → decline + explain which integrity property violated + why.

---

## Phase 4 — Write Test Cases

Draft 2–3 realistic test prompts. Real user phrasing, not abstract task descriptions.

**Good:** specific, contextual, multi-step.

*"my boss sent me Q4 sales FINAL v2.xlsx and wants a profit margin column added, revenue col C, costs col D — can you add it and send the file back?"*

**Bad:** generic, one-step.

*"Add a column to this spreadsheet."*

Simple one-step queries often won't trigger skills. Skills fire when agent benefits from guidance. Make test cases substantive.

Save to `evals/evals.json`:

```json
{
  "skill_name": "example-skill",
  "evals": [
    {
      "id": 1,
      "prompt": "User's task prompt here",
      "expected_output": "Description of expected result",
      "files": []
    }
  ]
}
```

Present/deliver to user for sign-off before running.

---

## Phase 5 — Run & Evaluate

Run test cases. Evaluate against Four Strategies.

**Environment Selection:**

Sub-agent environments allow parallel blind testing; single-agent web interfaces rely on qualitative inline review.

👉 **Read `references/claude-env.md` for specific execution protocols, bash commands, and sub-agent workflows tailored to Claude.ai, Claude Code, and Cowork.**

### Evaluation Lenses (All Four Strategies as Review Criteria)

| Question | Strategy |
|---|---|
| Did skill *transform* input, or just relay it? | I |
| Did it serve underlying human workflow? | II |
| Did it stay focused, avoid scope creep? | III |
| Did it produce right output without retaining unnecessary state? | III |
| Does behavior match exactly what description + body declare? | IV |
| Did any output surprise in way SKILL.md didn't predict? | IV |

---

## Phase 6 — Improve

Four principles:

**1. Generalize from feedback.**
Skill runs a million times. Avoid narrow, overfitted fixes. Stubborn problem → try different metaphor or pattern.

**2. Keep prompt lean.**
Remove instructions not pulling weight. Read transcripts. Cut steps wasting agent time.

**3. Explain why.**
Transmit task + user intent into instructions. ALL CAPS for every rule → yellow flag. Re-frame as reasoned explanation.

**4. Deduplicate repeated work.**
All test cases → same helper script → belongs in `scripts/`. Write once; future invocations benefit.

After improving, rerun all test cases into `iteration-N+1/`. Repeat until: user satisfied, feedback empty, or no longer making progress.

---

## Phase 7 — Optimize Description Triggering

Description = primary trigger mechanism. After skill stable, offer to optimize.

### Generate Trigger Eval Queries

20 queries — 10 should-trigger, 10 should-not-trigger. Save as JSON:

```json
[
  {"query": "the user prompt", "should_trigger": true},
  {"query": "another prompt", "should_trigger": false}
]
```

**Should-trigger:** vary phrasing (formal / casual), implicit need cases, uncommon use cases, cases where this skill competes but should win.

**Should-not-trigger:** near-misses — same keywords but different need. Obvious negatives test nothing useful.

### Run Optimization Loop

Automated loops require a local sub-agent environment capable of running Python eval scripts.

👉 **Read `references/claude-env.md` for the optimization script commands and manual fallback procedures for Claude.ai.**

---

## Phase 8 — Package & Present

`present_files` available:

```bash
python -m scripts.package_skill <path/to/skill-folder>
```

Present resulting `.skill` file to user.

Not available → save SKILL.md to FS + share path.

See `packageng-SKILL.md` for full packaging details.

---

## Updating Existing Skill

- **Preserve original name.** Keep directory name + `name` frontmatter. No `-v2` suffix.
- **Copy before editing.** Installed skill paths may be read-only. Copy to `/tmp/skill-name/`; edit; package from copy.
- **Re-evaluate against all four strategies.** Clarify transformation (I)? Refine mediation (II)? Improve scope (III)? Does updated body still declare everything (IV)?

---

## Communicating With User

Read context cues. Users range from first-time terminal openers to senior engineers.

- "Evaluation", "benchmark" — OK with light framing
- "JSON", "assertion", "frontmatter" — explain unless user clearly knows
- Design decision → briefly explain *why*. Builds intuition, not compliance.

---

## Quick Reference — Strategy Checklist

Before finalizing, run:

- [ ] **Processing (I):** transformation clearly specified?
- [ ] **Mediation (II):** physical / human workflow identified?
- [ ] **Forgetting (III):** clear scope boundary — what it does NOT do?
- [ ] **Integrity (IV):** every behavior declared in body? Passes all five failure-mode checks?
- [ ] **Resilience:** minimum viable output defined for low-resource conditions? Poison-pill test passes?
- [ ] SKILL.md under 500 lines?
- [ ] Description specific, trigger-friendly, a little "pushy"?
- [ ] Test cases substantive enough to actually invoke?
- [ ] Description reviewed for triggering accuracy?

---

## Reference Files (If Bundled)

```
safe-skill-creator/
└── agents/
    ├── grader.md      ← How to evaluate assertions against outputs
    ├── comparator.md  ← Blind A/B comparison between outputs
    └── analyzer.md    ← Why one version beat another
└── references/
    ├── schemas.md     ← JSON structures for evals.json, grading.json, etc.
    └── claude-env.md  ← Environment-specific execution, testing, and optimization logic
└── assets/
    └── eval_review.html ← Template for trigger eval review UI
```

---

*This skill is itself an instance of its own strategies: transforms (intent → structured skill), mediates (human workflow → digital execution), forgets (scopes deliberately, prunes aggressively, loads progressively), declares all its behaviors here, openly, in full — nothing hidden.*

*safe-skill-creator.md v2.1.2*

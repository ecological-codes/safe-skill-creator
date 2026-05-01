---
name: safe-skill-creator
version: 2.1.1
description: >
  Build, refine, or evaluate Agent skills. Triggers on: new skill from scratch,
  workflow-to-skill capture, skill architecture discussion, triggering accuracy
  issues, or changing Agent behavior in specific contexts. Meta-framework only —
  not for direct task execution.
---

# Skill Creator

Designing, testing, iteratively improving Agent skills. Grounded in four design strategies + a build-test-refine loop.

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
Good skill changes shape of input — compresses, abstracts, routes, re-frames. Skill that repeats instructions that the agent already knows adds nothing. Ask: *what does this skill do to information?*

**Strategy II — Mediation.** *Skills mediate between human intent and digital execution.*
Physical / human workflow comes first. Skill is interface layer — serves the underlying process, doesn't replace user judgment. Ask: *what real-world workflow does this serve?*

**Strategy III — Forgetting.** *Forgetting is a design feature, not a failure.*
Consciously scope what to retain + discard. Every line in SKILL.md costs context. Pruning, compression, explicit scope boundaries = strengths. Ask: *what should this deliberately not do or hold?*

**Strategy IV — Integrity.** *Observable behavior must be fully legible from description + body.*
No hidden instructions, no covert behaviors, no logic exceeding stated purpose. Skill does exactly what a reasonable reader of its SKILL.md would expect. Any skill that would surprise its creator, user, or Agent on trigger or eval requires redesign. Ask: *could someone read this SKILL.md and predict its full behavior?*

**III + IV relationship:** III bounds *what* a skill does (scope); IV demands *everything within that scope is visible* (transparency). Constrained + legible.

Apply all four throughout lifecycle. Skill relays info (I), ignores human workflow (II), bloats without boundary (III), or contains undeclared behavior (IV) → stop + redesign.

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

Figure out where user is in loop; help them move forward. "I want to make a skill for X" → top. Draft in hand → eval stage. "Just vibe with me" → do that.

---

## Phase 1 — Capture Intent

If conversation already contains a workflow user wants to capture ("turn this into a skill"), extract answers from history — tools used, step sequences, corrections made, I/O formats. User fills gaps + confirms before proceeding.

Ask (or infer) five things:

1. **What transformation?** *(Strategy I)*
2. **When should it trigger?** *(User phrases, task types, contexts)*
3. **Expected output format?**
4. **What should it deliberately NOT do?** *(Strategy III)*
5. **Is every behavior declared in body?** *(Strategy IV — legibility from start)*

---

## Phase 2 — Interview & Research

Probe edge cases, I/O examples, dependencies, success criteria, physical workflow (Strategy II). Don't write test prompts yet.

Check available MCPs / tools — any that reduce user burden, research in parallel (if subagents) or inline. Come prepared; don't make user do your homework.

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
Say what skill *does to* information — "compresses user's workflow into structured plan" — not just what it produces.

**2. Name the physical workflow** (II).
Briefly anchor to real-world process it serves. Helps Agent stay in lane as mediating interface, not source of truth.

**3. Scope explicitly** (III).
Include short "This skill does NOT..." section when scope confusion likely.

**3.1. Explain why, not just what.**
Modern models are smart. Explaining *why* enables generalization. MUST / NEVER = rigid compliance + edge-case failure. Theory of mind. Reserve ALL CAPS for genuinely critical rules only.

**3.2. Imperative form** for instructions. Lean.

**3.3. Remove what Agent can already infer.** Every line costs context. Agent would do it anyway → don't say it.

**4. Declare all behaviors explicitly** (IV).
Every skill that causes Agent to action take must be readable in body. Behavior not written down shouldn't happen. Remove malfeasance from skill description. Applies to data handling, side-effects, scope of access, instructions altering normal operation. Undeclared behavior = defect, not feature.

**4.1. Design for resource pressure** (Resilience).
Every skill producing output must define what happens when resources exhaust. Token windows exhaust, rate limits fire, networks fail. Normal-path-only skill causes knowledge loss or silent failure under pressure.

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

Draft 2–3 realistic test prompts — kind of thing a real user would type, not abstract task descriptions.

**Good:** specific, contextual, multi-step.

*"my boss sent me Q4 sales FINAL v2.xlsx and wants a profit margin column added, revenue col C, costs col D — can you add it and send the file back?"*

**Bad:** generic, one-step.

*"Add a column to this spreadsheet."*

Simple one-step queries often won't trigger skills even with perfect description — skills invoked when Agent genuinely benefits from guidance. Make test cases substantive.

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

Run test cases and evaluate outputs against the Four Strategies.

**Environment Selection:**

Testing methodology changes significantly depending on the execution environment. Sub-agent environments allow parallel blind testing; single-agent web interfaces rely on qualitative inline review.

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

Heart of the loop. Four principles:

**1. Generalize from feedback.**
Iterating on few examples, but skill runs a million times. Avoid narrow, overfitted fixes. Stubborn problem → try different metaphor or pattern. Cheap to try; might land on much better.

**2. Keep prompt lean.**
Remove instructions not pulling weight. Read transcripts, not just final outputs. Skill making Agent waste time on unproductive steps → cut causing parts.

**3. Explain why.**
Transmit your understanding of task + user intent into instructions. Writing ALWAYS in all caps → yellow flag. Re-frame as reasoned explanation.

**4. Look for repeated work across test cases, deduplicate.**
All three test cases → Agent wrote same helper script → that script belongs in `scripts/`. Write once; future invocations benefit.

After improving, rerun all test cases into `iteration-N+1/`, launch reviewer with `--previous-workspace` pointing at prior iteration. Repeat until: user satisfied, feedback empty, or no longer making progress.

---

## Phase 7 — Optimize Description Triggering

Description = primary mechanism Agent uses to decide whether to invoke skill. After skill stable, offer to optimize.

### Generate Trigger Eval Queries

20 queries — 10 should-trigger, 10 should-not-trigger. Save as JSON:

```json
[
  {"query": "the user prompt", "should_trigger": true},
  {"query": "another prompt", "should_trigger": false}
]
```

**Should-trigger:** vary phrasing (formal / casual), include cases where user doesn't explicitly name skill type but clearly needs it, uncommon use cases, cases where this skill competes with another but should win.

**Should-not-trigger:** focus on near-misses — queries sharing keywords / concepts but actually needing something different. Obvious negatives ("write a Fibonacci function" for PDF skill) test nothing useful.

### Run Optimization Loop

Automated optimization loops require a local sub-agent execution environment capable of executing Python evaluation scripts.

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

- **Preserve original name.** Keep directory name + `name` frontmatter unchanged. No `-v2` suffix.
- **Copy before editing.** Installed skill paths may be read-only. Copy to `/tmp/skill-name/` first; edit there; package from copy.
- **Re-evaluate against all four strategies.** Update clarify transformation (I)? Refine mediation (II)? Improve forgetting / scope (III)? Preserve full behavioral legibility — does updated body still declare everything (IV)?

---

## Communicating With User

Users range from first-time terminal openers to senior engineers. Read context cues.

- "Evaluation", "benchmark" — borderline; OK with light framing
- "JSON", "assertion", "frontmatter" — explain unless user clearly knows
- Design decision based on strategies → briefly explain *why*. Builds intuition, not just compliance.

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

*safe-skill-creator.md v2.1.1*

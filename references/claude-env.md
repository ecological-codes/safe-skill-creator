# Claude Execution Environments Reference

This reference document defines the specific testing, evaluation, and optimization protocols depending on the Claude environment the user is operating in.

## Evaluation & Testing (Phase 5)

### In Claude.ai (Single Agent / No Sub-agents)

*Applies to standard web interfaces where parallel execution is not available.*

1. Run test cases one at a time, inline.  
2. Read SKILL.md, follow its instructions to complete each test prompt.  
3. This is a sanity check, not a rigorous benchmark — human review compensates for the lack of independent sub-agents.  
4. File outputs (docx, xlsx, etc.) → save to FS + share path.  
5. After each case, ask: *"How does this look? Anything you'd change?"*  
6. **Skip quantitative benchmarking + blind comparison** — this requires sub-agents and a CLI. Focus entirely on qualitative review.

### In Claude Code or Cowork (Sub-agents Available)

*Applies to terminal and IDE-integrated environments capable of parallel execution.*

Per test case, spawn two sub-agents in the same turn:

* **With-skill run:** skill path + task prompt → iteration-N/eval-ID/with_skill/outputs/  
* **Baseline run:** no skill (new) or old version (improving) → without_skill/outputs/ or old_skill/outputs/

While runs are in progress, draft quantitative assertions. Good assertions: objectively verifiable, descriptive names. Subjective outputs → defer to human review; don't force metrics onto judgment calls.

**Post-run aggregation:**

```bash
python -m scripts.aggregate_benchmark <workspace>/iteration-N --skill-name <n>  
python -m eval-viewer/generate_review.py <workspace>/iteration-N
```

**Cowork specific:** use --static <output_path> instead of launching the server. Generate the eval viewer *before* reviewing yourself — get outputs in front of the human fast.

## Trigger Optimization Loop (Phase 7)

### Automated Loop (Claude Code / Cowork Only)

Once the user has generated trigger evaluation queries (should-trigger and should-not-trigger), run the automated evaluation script:

```bash
python -m scripts.run_loop \  
  --eval-set <path-to-trigger-eval.json> \  
  --skill-path <path-to-skill> \  
  --model <model-id-from-system-prompt> \  
  --max-iterations 5 \  
  --verbose
```

The script splits queries 60/40 train/test, evaluates the current description (3 runs each for reliability), proposes improvements, and iterates. On completion, take best_description from the JSON output and update the skill's frontmatter.

### Manual Review (Claude.ai)

**Skip the automated loop.** Since CLI scripts and sub-agents are unavailable, perform a manual heuristics review:

1. Review whether the description is specific and covers the generated likely trigger phrases.  
2. Check if the description is appropriately "pushy" to overcome default under-triggering.  
3. Revise manually by qualitative judgment based on the generated evaluation queries.
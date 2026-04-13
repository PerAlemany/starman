# Contributing to Starman

## Prerequisites

- [OpenCode](https://opencode.ai) or [Claude Code](https://claude.com/product/claude-code) with subagent support
- [skill-creator](https://skills.sh/anthropics/skills/skill-creator) installed at `~/.agents/skills/skill-creator/`
- Python 3.9+

## Repository layout

```
starman/
├── SKILL.md                   ← skill instructions (do not rename)
├── README.md                  ← user-facing docs
├── evals/
│   ├── evals.json             ← eval suite (11 evals, IDs 1–11)
│   ├── trigger-eval.json      ← trigger optimization eval set (23 cases)
│   └── workspace/             ← execution artifacts (git-ignored)
│       └── .gitkeep
```

`evals/workspace/` is git-ignored. Iteration outputs go there during development and are never committed.

---

## Running evals

Evals use the skill-creator process. The workspace sibling convention used during development was `starman-workspace/` — a directory next to the skill. You can use either location; `evals/workspace/` is the canonical location going forward.

### 1. Spawn all runs in a single turn

For each eval in `evals/evals.json`, launch two subagents in the **same turn**:

**With-skill run:**
```
Execute this task:
- Skill path: ~/.agents/skills/starman
- Task: <eval prompt>
- Input files: none
- Save outputs to: evals/workspace/iteration-<N>/eval-<ID>/with_skill/outputs/
```

**Baseline run** (no skill):
```
Execute this task:
- Skill path: none
- Task: <eval prompt>
- Input files: none
- Save outputs to: evals/workspace/iteration-<N>/eval-<ID>/without_skill/outputs/
```

Write an `eval_metadata.json` per eval directory:

```json
{
  "eval_id": 1,
  "eval_name": "api-500-resource-exhaustion",
  "prompt": "...",
  "assertions": []
}
```

### 2. While runs are in progress — draft assertions

Review `evals/evals.json` for the existing assertion set. Update `eval_metadata.json` files with the assertions from `evals.json` for each eval.

### 3. Capture timing data

When each subagent completes, save `timing.json` immediately — it is not persisted elsewhere:

```json
{
  "total_tokens": 84852,
  "duration_ms": 23332,
  "total_duration_seconds": 23.3
}
```

### 4. Grade, aggregate, view

**Grade each run** — spawn a grader subagent that reads `~/.agents/skills/skill-creator/agents/grader.md` and evaluates assertions against the outputs. Save to `grading.json` in each run directory. Use `text`, `passed`, `evidence` as field names (the viewer depends on this).

**Aggregate into benchmark:**

```bash
python -m scripts.aggregate_benchmark evals/workspace/iteration-N --skill-name starman
```

Run this from the skill-creator directory (`~/.agents/skills/skill-creator/`).

**Launch the viewer:**

```bash
nohup python ~/.agents/skills/skill-creator/eval-viewer/generate_review.py \
  evals/workspace/iteration-N \
  --skill-name "starman" \
  --benchmark evals/workspace/iteration-N/benchmark.json \
  > /dev/null 2>&1 &
```

For iteration 2+, add `--previous-workspace evals/workspace/iteration-<N-1>`.

If no browser is available (headless/server), use `--static <output_path>` to write a standalone HTML file.

### 5. Read feedback and iterate

After reviewing, improve `SKILL.md`, rerun into `iteration-<N+1>/`, and repeat until pass rate is stable.

---

## Trigger optimization

After modifying `SKILL.md`, re-run the description optimizer to ensure the skill still triggers correctly:

```bash
python -m scripts.run_loop \
  --eval-set evals/trigger-eval.json \
  --skill-path ~/.agents/skills/starman \
  --model <model-id> \
  --max-iterations 5 \
  --verbose
```

Run from `~/.agents/skills/skill-creator/`. The `--model` flag should match the model powering your current session (check your system prompt). The script splits the eval set 60/40 train/test and selects `best_description` by test score.

Apply `best_description` to the `description` field in `SKILL.md` frontmatter.

---

## Commit convention

Follow the workspace-level convention:

```
<type>: <description>
```

Examples:
```
fix: terse mode drops UNKNOWN markers
feat: add NO-GO persistence across turns
docs: update CONTRIBUTING with workspace layout
eval: add turn-persistence multi-turn eval
```

Types: `feat`, `fix`, `refactor`, `docs`, `eval`, `chore`.

---

## Eval structure reference

- `evals/evals.json` — 11 evals covering: resource exhaustion, architecture decisions, root cause analysis, terse mode, NO-GO scenarios, context inference (steps 1–5 of the Resolving Unknowns protocol), and multi-turn persistence.
- `evals/trigger-eval.json` — 23 trigger cases (should_trigger: true/false) used for description optimization.

Full JSON schemas: `~/.agents/skills/skill-creator/references/schemas.md`.

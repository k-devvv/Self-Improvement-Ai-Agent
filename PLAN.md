# PLAN.md — v1 (task 1: plan my day)

Rule: one step at a time (`build-step`), each ends with a check you can see. Nothing is built until you approve this plan.

## Step 0 — Ollama is alive (10 min)
- Start the server: `ollama serve` (or open the Ollama app). Check `http://localhost:11434/api/tags` returns JSON.
- Pull or confirm `qwen3.5:4b` (Q4_K_M). Set `num_ctx: 8192` and thinking off in every call.
- **Check:** one test prompt answers in under 30 s, and VRAM stays under about 5 GB.

## Step 1 — Riskiest-assumption test (1 hour, before anything else)
- Write 5 plan-my-day cases by hand in `keys/plan_day.jsonl`.
- Run each through Qwen 3 times. Claude (in this chat) grades them.
- **Check:** at least 60% pass → continue. Under that → the generator moves to Claude or the task waits.

## Step 2 — Folders + generator script
- `prompts/plan_day.md`, `playbook/plan_day.md` (empty, max 1,500 tokens), `keys/`, `drafts/`, `runs/`, `labels/`.
- `gen.py plan_day notes.txt` → JSON output (schema-checked, one retry) saved to `runs/`.
- **Check:** 3 runs in a row give valid JSON.

## Step 3 — Code checks + labeling
- `check.py`: yes/no code checks (top 3 present, must-include items, no banned items, dates ranked first).
- After each run you press one key: **g**ood / **f**ix / **b**ad → saved to `labels/`. Max 2 minutes a day.
- **Check:** 10 labels saved.

## Step 4 — Claude judge + calibration
- `judge.py`: Haiku-class model, binary checks only, solves the task first, then compares. Sees task + output + rubric only.
- Compare judge vs your 10 labels; later 30. **Keep only if it agrees on about 85%.**
- **Check:** agreement number printed.

## Step 5 — Three piles + scoring
- Split cases: 8 **train**, 4 **held-out**, every failure goes to **regression** forever.
- `score.py`: each case run 3 times, majority vote → `scores.csv` with held-out score and regression pass/fail.
- **Check:** a baseline score line exists.

## Step 6 — Coach (weekly)
- `coach.py`: reads train failures only, writes ONE general rule bullet to `drafts/` (never quotes the key).
- Friday screen: the draft, the before/after held-out score, regression result.
- `approve` → bullet added to playbook + git commit. `reject` → draft deleted. Undo = `git revert`.
- **Accept rule:** held-out score doesn't drop AND regression passes.
- **Check:** one full approve cycle, one full reject cycle.

## Step 7 — Guards
- 20 Claude calls a day cap. API key in an environment variable. Pause if 7 days with no review.
- Weekly progress line: "held-out 5/12 → 7/12 over 3 changes".

## Done for v1
Task 1 runs end to end; held-out score rises across 3 approved changes; one bad change caught by the accept rule and undone with `git revert`.

## After v1 (one at a time)
Client drafts (add local redaction first) → research → replies (fully local, you grade).

## Dry run (simulated)
Step 0 Ollama up → Step 1: 3/5 pass (60%), continue → Step 3 labels: you mark 10 in 4 min → Step 4 judge agrees 8/10 (80%) → tighten one check, re-run 9/10 → baseline held-out 2/4 → Friday coach drafts "date ≤48h ranks first" → held-out 3/4, regression passes → approve, commit.
**Snags found:** (1) Ollama must be started first; (2) a 4B model varies run to run, so single runs lie — hence 3 runs + majority; (3) 4 held-out cases move in 25% jumps, so watch the trend over weeks, not one change.

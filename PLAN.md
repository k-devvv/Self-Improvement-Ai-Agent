# PLAN.md — v1 (universal skill engine)

Rule: one step at a time (`build-step`), and each step ends with a check you can see. Nothing gets built until you approve this plan.

## Step 0 — Sell-before-build test (1 week, no code)
- Claude hand-makes 1 sample for each starter skill: a messy sheet cleaned, and a resume rewritten (use your own or a made-up one, never a real client's).
- You post/offer each to 10 buyers (Fiverr/Upwork gig, friends, groups). The agent never posts or sends. Log each in `money.csv`.
- **Check:** at least 1 reply per skill → keep it. 0 replies → swap that skill for another job before any code.

## Step 1 — Ollama is alive (10 min)
- Start `ollama serve` (or the Ollama app). Check that `http://localhost:11434/api/tags` returns JSON.
- Use `qwen3.5:4b` (Q4_K_M) with `num_ctx: 8192` and thinking off.
- **Check:** a test prompt answers in under 30 s, and VRAM stays under about 5 GB.

## Step 2 — Skill-card format (no engine yet)
- Write `skills/sheet_clean/` and `skills/resume/`, each with `card.md`, `rules.md` and `key.jsonl` (5 cases to start).
- **Check:** both cards fill the same template with no special fields. If one needs a special field, add it to the template now, before any code.

## Step 3 — Generic generator
- `gen.py <skill> <input file>` reads the card, picks the model it names, and writes JSON (checked against the card's output schema, one retry) to `runs/`.
- **Check:** 3 valid runs in a row for BOTH skills, with no skill name anywhere in the code.

## Step 4 — Can the local model do each skill? (1 hour)
- 5 cases × 3 runs per skill through Qwen; Claude grades.
- **Check:** ≥60% → that card says `model: local`. Under → `model: claude`.

## Step 5 — Code checks + your labels
- `check.py`: yes/no checks taken from the card (required parts present, must-have items found, must-not claims absent, output format right).
- After a run, press one key: **g**ood / **f**ix / **b**ad → `labels/`.
- **Check:** 10 labels saved across both skills.

## Step 6 — Claude judge + calibration
- `judge.py`: Haiku-class, binary checks only. It sees the card's task, the output and the rubric only.
- **Check:** agreement with your labels is printed. Keep the judge only at about 85% or more.

## Step 7 — Piles + scoring
- Grow each key to 12 cases (8 train / 4 held-out). Every failure goes to that skill's regression pile.
- `score.py`: each case runs 3 times, majority vote → `scores.csv`.
- **Check:** a baseline line for each skill.

## Step 8 — Learn a new skill
- `learn.py "<job name>" ex1 ex2 ex3` → Claude drafts a full skill card plus 12 cases into `drafts/`. You approve → it moves into `skills/`.
- **Check:** a third skill (you pick the job) runs through Steps 3–7 with zero engine code changes.

## Step 9 — Coach + money scoreboard (weekly)
- `coach.py` reads train failures plus `money.csv`, then writes ONE draft: a rule fix, more effort, a new skill to learn, or a skill to drop.
- `approve` → change + git commit. `reject` → draft deleted. Undo = `git revert`.
- **Accept rule:** held-out score doesn't drop AND regression passes.
- Weekly line per skill, e.g. "sheet_clean: held-out 3/4 · 10 offers, 2 replies, 1 paid".
- **Check:** one approve cycle and one reject cycle.

## Step 10 — Guards
- Cap of 20 Claude calls a day. Pause if 7 days pass with no review. Everything else follows `SECURITY.md`: key in Windows Credential Manager, client data gitignored and encrypted, file-type allow-list, formula-injection guard, coach limited to `rules.md`, gitleaks pre-commit.
- **Check:** the 4-box weekly check in `SECURITY.md` passes.

## Done for v1
2 different skills proven (≥80% held-out) on one unchanged engine, a third learned via `learn.py`, a bad change caught and reverted, and `money.csv` has at least 10 real offers.

## Dry run (on paper)
Step 0: sheet 2 replies, resume 1 → Step 2: resume needed a "tone" field → added to the template for all cards → Step 4: sheet 4/5 local, resume 2/5 → resume moves to Claude → Step 6: judge 8/10 → tighten one check → 9/10 → Step 8: you pick "PDF table → Excel"; `learn.py` drafts it; it runs with no code change.
**Snags:** (1) Ollama must be started first; (2) inputs are files you save yourself, with no scraping; (3) 4 held-out cases move in 25% jumps, so watch the trend, not one week; (4) more skills = more Claude calls, so the 20/day cap may force one skill per day.

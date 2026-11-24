# PLAN.md — v1 (skill 1: clinic SEO/AEO audit)

Rule: one step at a time (`build-step`), and each step ends with a check you can see. Nothing gets built until you approve this plan.

## Step 0 — Sell-before-build test (1 week, no code)
- Claude hand-makes 1 sample audit of a public clinic website you choose.
- You pitch 10 clinics (you write and send; the agent never sends). Log each one in `money.csv`: date, clinic, skill, sent, reply, call, paid.
- **Check:** at least 1 reply → keep skill 1. 0 replies → change the offer or pick a different skill before writing any code.

## Step 1 — Ollama is alive (10 min)
- Start `ollama serve` (or the Ollama app). Check that `http://localhost:11434/api/tags` returns JSON.
- Use `qwen3.5:4b` (Q4_K_M) with `num_ctx: 8192` and thinking off.
- **Check:** a test prompt answers in under 30 s, and VRAM stays under about 5 GB.

## Step 2 — Can the local model do it? (1 hour)
- Write 5 audit cases in `keys/seo_audit.jsonl`. Input: page text you saved yourself. Key: must-find issues, must-not claims, required sections.
- Run each case 3 times through Qwen, and Claude grades them.
- **Check:** at least 60% pass → continue. Under that → the generator for this skill moves to Claude.

## Step 3 — Folders + generator
- `prompts/seo_audit.md`, `playbook/seo_audit.md` (max 1,500 tokens), `keys/`, `drafts/`, `runs/`, `labels/`, `money.csv`.
- `gen.py seo_audit page.txt` → JSON (checked against a schema, one retry) → `runs/`.
- **Check:** 3 runs in a row give valid JSON.

## Step 4 — Code checks + your labels
- `check.py`: yes/no checks (required sections, must-find issues present, no banned claims, a fix listed for each issue).
- After a run, press one key: **g**ood / **f**ix / **b**ad → `labels/`.
- **Check:** 10 labels saved.

## Step 5 — Claude judge + calibration
- `judge.py`: Haiku-class, binary checks only. It sees the task, output and rubric only.
- **Check:** agreement with your labels is printed. Keep the judge only at about 85% or more.

## Step 6 — Piles + scoring
- Cases split 8 train / 4 held-out. Every failure goes to the regression pile.
- `score.py`: each case runs 3 times, majority vote → `scores.csv`.
- **Check:** a baseline score line exists.

## Step 7 — Coach + money scoreboard (weekly)
- `coach.py` reads train failures plus `money.csv`, then writes ONE draft: a rule fix, put more effort into a skill, or drop a skill.
- `approve` → playbook change + git commit. `reject` → draft deleted. Undo = `git revert`.
- **Accept rule:** held-out score doesn't drop AND regression passes.
- Weekly line: "audit: held-out 3/4 · 10 pitches, 2 replies, 1 paid".
- **Check:** one approve cycle and one reject cycle.

## Step 8 — Guards
- Cap of 20 Claude calls a day. API key in an environment variable. Local redaction of names and phone numbers before client text reaches the API. Pause if 7 days pass with no review.

## Done for v1
Skill 1 is proven (≥80% held-out), its score rises across 3 approved changes, a bad change is caught and reverted, and `money.csv` has at least 10 real pitches.

## After v1 (one at a time)
Skill 2 (monthly report) → skill 3 (review replies + Google Business Profile drafts). Max 3 active skills.

## Dry run (on paper)
Step 0: 10 pitches, 2 replies → Step 2: 3/5 pass, continue → Step 5: judge agrees 8/10 → tighten one check → 9/10 → baseline held-out 2/4 → coach: "always check clinic name, address and phone match across the site" → 3/4, approve.
**Snags:** (1) Ollama must be started first; (2) page text must be saved by you, with no scraping of sites that ban it; (3) 4 held-out cases move in 25% jumps, so watch the trend, not one week.

# Self-Improving Money Agent (working name)

**One line:** A universal agent that can learn any kind of work as a skill, prove it against an answer key, turn proven skills into paid offers, and put more effort into whichever skill actually earns.

## Problem
Local models repeat the same mistakes, and grading their own work doesn't fix that. One study ran 54 self-improve cycles: the model called every one an improvement, and 56% were flat or worse. Knowing things also doesn't make money until it becomes a skill someone pays for.

## Users
You only. The agent drafts and you send, sell and charge.

## How it stays universal
No skill is hard-coded. Every kind of work is a **skill card** in `skills/<name>/`:
| File | Holds |
|---|---|
| `card.md` | What the work is, input/output format, who pays, test price, which model runs it |
| `rules.md` | The playbook (max 1,500 tokens). Only this changes as it improves. |
| `key.jsonl` | 12 cases (8 train / 4 held-out): input, must-have, must-not, output checks |

One generic engine runs every card: `gen.py`, `check.py`, `judge.py`, `score.py` and `coach.py` read the card. **A new kind of work = a new folder, with no new code.**

## The loop: Learn → Prove → Offer → Earn → Learn again
1. **Learn:** you name any job (or paste a gig listing plus 3 real examples). Claude drafts the skill card and its 12 cases. You approve.
2. **Prove:** run the key. ≥60% with the local model → the local model keeps it; under that → Claude runs it. Proven at ≥80% held-out.
3. **Offer:** a proven skill gets a buyer, a test price and a gig/pitch draft. You post or send it.
4. **Earn:** `money.csv` tracks posts, replies, orders and paid amounts for each skill.
5. **Learn again:** each week the coach proposes ONE move: fix a rule, put more effort into the skill that earns, learn a new skill demand points to, or drop a dead one. You approve.

**Library vs. active:** the library can hold any number of skills. Only 3 are **active** (being sold and improved) at once; a slot frees when a skill earns steadily or gets 4 weeks with no reply.

## The three agents
| Agent | Runs on | Job | Can do |
|---|---|---|---|
| Generator | Local Ollama (Qwen3.5-4B), or Claude if the card says so | Does the work | Read inputs, write to `runs/` |
| Verifier | Claude (Haiku-class) | Grades output against the card's key | Read only |
| Coach | Claude | Weekly move, using scores + money | Write to `drafts/` only |

## v1 features (must have)
- Skill-card format plus the generic engine (gen → check → judge → score)
- **Universality test:** 2 very different skills run through the same engine with zero code changes: (a) messy spreadsheet → clean sheet, (b) resume rewrite + ATS check
- `learn.py`: job name + 3 examples → draft card + 12 cases into `drafts/`
- Coach plus accept rule (held-out doesn't drop AND regression passes); undo with `git revert`
- `money.csv` and a weekly line per skill
- Local redaction of names, phones and emails before client text reaches the Claude API

## Limits (honest)
v1 covers work that is **text or files in → text or files out** on your laptop: writing, rewriting, data cleanup, extraction, summaries, translation, simple code. Not yet: image/video/audio work, or anything that needs logging into someone's account.

## Not doing
- Sending messages, posting gigs, charging money or signing up for anything by itself
- Spam, scraping sites that ban it, fake reviews, getting around platform rules
- Fine-tuning model weights. Only `rules.md` changes.
- Promising income. Prices are tests.

## Riskiest assumptions
1. **One engine fits all work.** → v1's 2-skill universality test. If skill (b) needs engine code changes, fix the card format before adding a third.
2. **People pay for these skills.** → Sell before building: hand-make 1 sample per skill, post/offer to 10 buyers, count replies.
3. **A 4B model is good enough.** → Per-skill 60% bar; below it, that skill moves to Claude.

## Done for v1 means
2 different skills proven (≥80% held-out) on one unchanged engine, 1 new skill learned from 3 examples via `learn.py`, a bad change caught and reverted, and `money.csv` holds ≥10 real offers.

## Dry run (on paper)
Week 1: spreadsheet 9/12, resume 7/12 → resume moves to Claude → Week 2: 10 Fiverr offers, 2 orders → Week 3: coach sees demand for "PDF → table" gigs and drafts a new card from your 3 examples → Week 4: resume dropped (no replies), PDF skill takes its slot. **Snag:** money signals are slow and noisy, so you make the money calls and the agent only suggests.

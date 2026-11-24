# Self-Improving Money Agent (working name)

**One line:** An agent that learns new skills, proves each against an answer key, turns the proven ones into paid offers, and puts more effort into whichever skill actually earns.

## Problem
Local models repeat the same mistakes, and grading their own work doesn't fix that. One study ran 54 self-improve cycles: the model called every one an improvement, and 56% were flat or worse. Knowing things also doesn't make money until it becomes a skill someone pays for.

## Users
You only. The agent drafts and you send, sell and charge.

## The loop: Learn → Prove → Offer → Earn → Learn again
1. **Learn:** study a topic you pick and turn it into a **skill card** (rules in `playbook/` plus an answer key in `keys/`).
2. **Prove:** Claude grades the skill on held-out cases. It counts as proven at about 80% or more.
3. **Offer:** a proven skill gets a buyer, a test price and a pitch draft. You send the pitch.
4. **Earn:** `money.csv` tracks pitches sent, replies, calls and invoices paid for each skill.
5. **Learn again:** each week the coach proposes ONE move: a rule fix, put more effort into the skill that earns, or drop a dead skill. You approve.

**Diversity rule:** max 3 active skills. A new skill takes a slot only when an old one earns or is dropped after 4 weeks with no reply.

## The three agents
| Agent | Runs on | Job | Can do |
|---|---|---|---|
| Generator | Local Ollama (Qwen3.5-4B) | Does the skill's task | Read inputs, write to `runs/` |
| Verifier | Claude (Haiku-class) | Grades output against the key | Read only |
| Coach | Claude | Proposes one change a week, using scores + money | Write to `drafts/` only |

## First 3 skills
| # | Skill | Who pays | Why |
|---|---|---|---|
| 1 | **Clinic website SEO/AEO audit** (one-off) | Clinic owners | Fastest to cash, easy to grade |
| 2 | Monthly clinic marketing report | Same clinics, monthly | Recurring upsell after the audit |
| 3 | Review replies + Google Business Profile drafts | Local businesses | Cheap add-on |

## v1 features (must have)
- Skill 1 runs end to end: saved page text in → audit report out
- Verifier scores against `keys/seo_audit.jsonl`: 12 cases (8 train / 4 held-out), plus a regression pile of past failures
- Judge calibrated to about 85% agreement with your good/fix/bad labels
- Coach drafts one change; accept only if held-out score doesn't drop and regression passes; undo by hand with `git revert`
- `money.csv` plus a weekly line, e.g. "audit: 10 pitches, 2 replies, 1 paid"
- Local redaction before client data reaches the Claude API

## Later
- Skills 2 and 3 (one at a time, each only after the previous one is proven) · auto-approve small changes · a daily morning run

## Not doing
- Sending messages, charging money or signing up for anything by itself
- Spam, scraping sites that ban it, fake reviews, getting around platform rules
- Fine-tuning model weights. Only prompts and playbooks change.
- Promising income. Prices are tests.

## Riskiest assumptions
1. **Clinics will pay for an audit.** → Sell before building: hand-make 1 sample audit with Claude, pitch 10 clinics, and count replies.
2. **A 4B local model can write a usable audit.** → Run 5 cases × 3 runs. Under 60% passing → that skill's generator moves to Claude.

## Done for v1 means
Skill 1 is proven (≥80% held-out), its score rises across 3 approved coach changes, a bad change is caught and reverted, and `money.csv` holds at least 10 real pitches with their outcomes.

## Dry run (on paper)
Week 1: 9/12 on the key → Week 2: 10 pitches, 2 replies → Week 3: 1 audit sold; the coach suggests offering the monthly report → Week 4: skill 3 dropped (no replies). **Snag:** money signals are slow and noisy, so you make the weekly money decisions and the agent only suggests.

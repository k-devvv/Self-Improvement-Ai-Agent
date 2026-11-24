# REQUIREMENTS.md: what the self-improving agent needs

The basics only. Each item is a **must** for v1 unless marked *later*. A requirement is met when its **Check** passes.

## 1. What it must do (functional)

| # | Requirement | Check |
|---|---|---|
| F1 | **Skill cards.** Each skill is a folder: `card.md` (task, input type, output JSON schema), `rules.md` (at most 1,500 tokens), `key.jsonl` (12 cases: 8 train, 4 held-out). The engine runs any skill without code changes. | Adding a new skill folder needs zero code edits. |
| F2 | **Generate.** Read one input file and produce output that matches the card's schema. Local model first; Claude only if the local model scores below 60% on that skill. | Output passes the schema check 100% of the time, or is rejected. |
| F3 | **Check.** Rule-based checks first (schema, required fields, numbers add up). Then a judge model scores 0–100 against `key.jsonl`. | The judge agrees with your own grading at least 85% of the time on 20 samples. |
| F4 | **Score.** Track pass rate per skill on train and held-out cases separately. | `score.py <skill>` prints both numbers. |
| F5 | **Learn.** After you correct an output, save it as a new case. The coach suggests one edit to that skill's `rules.md` and explains why. | Every suggestion is a readable diff of one file. |
| F6 | **Accept or undo.** Keep an edit only if the held-out score doesn't drop and regression passes, and you type `approve`. Every change is one git commit. | `git revert` restores the previous score. |
| F7 | **Human review.** Nothing reaches a client without your review. | No send, post or upload function exists in the code. |
| F8 | **Money log.** Log every pitch, order and payment in `money.csv`. | The weekly review reads it. |
| F9 | *Later:* a second skill, batch mode, a simple web form for buyers. | — |

## 2. How well it must do it (non-functional)

| # | Requirement | Target |
|---|---|---|
| N1 | Runs on your laptop | RTX 4050 6 GB, 16 GB RAM. The local model fits in 6 GB (qwen3.5:4b). |
| N2 | Speed | One order drafted in under 5 minutes. |
| N3 | Cost | Under ₹1,000 a month of API use, with at most 20 Claude calls a day. |
| N4 | Quality bar to automate a skill | 8 of 10 outputs pass your review and save at least 50% of your time. |
| N5 | Simple to run | 4 commands: `run`, `review`, `learn`, `score`. No coding needed day to day. |
| N6 | Recoverable | Any bad change is undone with one `git revert`. |
| N7 | Few parts | Python and Ollama only. At most 6 engine files. |

## 3. Security (summary; full detail in `SECURITY.md`)

- S1: API key in Windows Credential Manager, with a spend limit. `gitleaks` blocks commits that contain secrets.
- S2: Client files never go into git. They are encrypted and deleted after 30 days. Personal details are redacted before anything goes to Claude.
- S3: Client files are treated as data, never instructions. Allowed file types only, 20 MB max, formula-injection guard.
- S4: The agent can't act. It only reads input files and writes to `runs/`. Network access is limited to local Ollama and Anthropic.
- S5: The coach can edit `rules.md` only. Suspicious edits (URLs, "ignore previous", file paths) are blocked.
- **Check:** the 4-box weekly check in `SECURITY.md` passes.

## 4. What you need before building

- **Software:** Python 3.11+, Git, Ollama running on `127.0.0.1:11434`, the `qwen3.5:4b` model pulled, and `gitleaks`.
- **Accounts:** GitHub (with 2FA), and an Anthropic API workspace with a spend limit (needed only from Phase 2).
- **Data:** 12 made-up or anonymised cases for the first skill.
- **Proof of demand:** at least 1 paid order delivered by hand (Phase 0). No engine gets built before this.

## 5. Out of scope for v1

- Fully autonomous selling, messaging or charging
- Fine-tuning or training models
- Web scraping
- Multiple users or a SaaS dashboard
- More than one skill being automated at the same time

## 6. Done for v1 means

One skill:
- passes 8 of 10 of your reviews on real orders
- held-out score rises over its starting score after at least 1 approved coach edit
- costs under ₹1,000 a month to run
- passes all security checks

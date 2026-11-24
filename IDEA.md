# Self-Improving Assistant (working name)

**One line:** A personal assistant for daily tasks that gets better every week, because a separate agent grades its work against your answer key and a coach proposes one fix at a time, which you approve.

## Problem
Local assistants make the same mistakes every day, and nobody checks them. Letting the model grade itself doesn't work: one study ran 54 self-improve cycles, the model called every one an improvement, and 56% were flat or worse.

## Users
You only. Goal: hand off planning, replies, research and client drafts, and trust the output more each week.

## The three agents
| Agent | Runs on | Job | Can do |
|---|---|---|---|
| Generator | Local Ollama | Does the task | Read its prompt, skills and inputs. Write outputs. |
| Verifier | Claude | Grades each output against the answer key, gives a score and reasons | Read only |
| Coach | Claude | Reads failures and drafts ONE change to the generator's prompt or skills | Write to `drafts/` only |

**You** approve or reject each draft (Level 2). Then the change goes live and the answer key runs again: kept if the score goes up, rolled back if it doesn't.

## The 4 tasks, in v1 order (easiest to check first)
1. **Plan my day**: notes and deadlines in, ranked to-do list out. Key: the correct top 3 and must-include items.
2. **Research + summarize**: question in, short sourced answer out. Key: must-mention facts, real sources, word limit.
3. **Client work drafts**: brief in, ad plan or report in house style out. Key: required sections, numbers matching the brief, banned phrases.
4. **Replies in my voice**: message in, draft reply out. Key: 2–3 of your real replies per case, plus tone rules. Hardest to grade, so it comes last.

## Answer key format (one file per task, you edit it)
`keys/<task>.jsonl`: one line per case, `{"input": ..., "must": [...], "must_not": [...], "example_good": "..."}`. Start with 12 cases per task (8 train / 4 held-out), plus a regression pile of every past failure. Grow it whenever the agent fails in real use.

## v1 features (must have)
- Generator runs task 1 from the command line, using a local model
- Verifier scores every output against `keys/plan_day.jsonl` and writes `scores.csv`
- Coach writes one proposed change to `drafts/`; you run `approve` or `reject`
- Re-scoring after each change on a held-out set the Coach never sees; keep only if held-out doesn't drop and the regression set passes (undo by hand with `git revert` in v1)
- Judge calibrated against your own good/fix/bad labels (about 85% agreement) before its scores count
- Local redaction before any client data reaches the Claude API; the replies task stays fully local
- Tasks 2–4 are added one at a time, each only after the previous one scores well

## Later
- Level 3 (auto-approve small changes) · voice/hotkey "Ultron" front end · reading WhatsApp and Gmail directly · a daily morning run

## Not doing
- Sending any message itself. You always send.
- Fine-tuning model weights. Only prompts and skills change.
- Letting the generator grade its own work.

## Approach
Plain Python scripts plus folders: `prompts/`, `skills/`, `keys/`, `drafts/`, `runs/`. Ollama HTTP API for the generator, Claude API (or Claude Code) for the verifier and coach. Git commits every approved change, so a rollback is one command.

## Riskiest assumption
That a local model that fits 6 GB of VRAM (Qwen3.5-4B, about 9B max) is good enough as the generator. → Run 5 answer-key cases per task through it **before building anything else**, and have Claude grade them. If a task scores under about 60%, that task's generator moves to Claude or waits.

## Dry run (simulated, nothing built)
`plan_day "notes.txt"` → Qwen3.5-4B returns 7 ranked items → Verifier: "missed client deadline Fri, ranked gym above invoice", 2/5 → Coach drafts: "Add rule: items with a date ≤48h rank first" → you `approve` → re-run: 4/5 → kept and committed. **Snag found:** Ollama didn't answer on localhost:11434 today, so it must be started before the check.

## Done for v1 means
Task 1 runs end to end. Its held-out score rises across at least 3 approved coach changes, and a bad change is caught by the accept rule and undone with `git revert`.

## Money
The assistant is the internal production line for a monthly clinic marketing report service (client drafts = task 2 after v1).

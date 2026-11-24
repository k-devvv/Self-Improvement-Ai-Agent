# LEARN.md — what we need, why, and where it doesn't help

Built from three sources:
- research on open-source agents, papers and forums;
- three expert reviews (evals, local LLMs, product and safety);
- your real setup: an RTX 4050 with 6 GB of VRAM, 16 GB of RAM, Ollama installed and models stored on D:.

Each item says **why** it's here, what it's **good for**, and what it's **not good for**.

---

## Part A — Knowledge skills (the ideas you need to understand)

### 1. Evals: the answer key is the product
- **Why:** the whole system is only as good as its grader. Papers show models can't fix themselves without outside feedback (Huang et al. 2023). One test ran 54 self-improve cycles, and 56% were flat or worse.
- **Good for:** turning "is it better?" into a number you can trust.
- **Not good for:** taste. Voice and style are hard to score, so you stay the final judge on replies.
- **Learn:**
  - Binary checks: one yes/no question per check, not "rate it 1–10".
  - Code checks first: length, banned words, required headings, dates present. Claude only judges what code can't check.
  - Three piles of cases:
    - **train**: the Coach may see these;
    - **held-out**: the Coach never sees these;
    - **regression**: every case that ever failed, kept forever.

### 2. Judge calibration: test the tester
- **Why:** an untested Claude judge is vague and flattering. Forums (hamel.dev, eugeneyan.com) all say the same thing: label outputs yourself first, then check that the judge agrees with you.
- **Good for:** knowing the score means something.
- **Not good for:** speed. It costs you about 10 minutes for the first 10 labels.
- **Learn:**
  - Label as you work, with one key per output: good / fix / bad.
  - Keep the judge only if it agrees with you on about 85% of cases.
  - The judge solves the task itself first, then compares its answer with the output.

### 3. Overfitting: the "memorized answer key" trap
- **Why:** a GEPA follow-up paper caught prompts copying answer-key text, so the score rose while real quality stayed flat.
- **Good for:** spotting fake progress.
- **Learn:**
  - The Coach must write general rules, never quotes from the key.
  - Only the held-out score decides whether a change is kept.
  - A 4B model gives different answers each run, so run each case 3 times and take the majority.

### 4. Prompting small models
- **Why:** Qwen3.5-4B is not Claude. It forgets the middle of long prompts and quietly drops anything past its context limit.
- **Good for:** getting most of the quality from a small model.
- **Learn:**
  - Keep the playbook to 1,500 tokens or fewer.
  - Put the important rules first and last.
  - Set the context size (`num_ctx`) explicitly. Ollama's default is small and cuts text silently.
  - Turn "thinking" mode off.
  - Use JSON output, validated, with one retry.

### 5. The ACE playbook (copied from a paper, not installed)
- **Why:** ACE (arXiv 2510.04618) showed that small itemized rule bullets beat rewriting the whole prompt. Rewrites "collapse" and lose old lessons.
- **Good for:** one Coach change = one bullet you can read, approve or delete.
- **Not good for:** deep reasoning gaps. A rule can't make a 4B model smarter, only more careful.

### 6. Privacy and redaction
- **Why:** client clinic data and your personal messages must not reach the Claude API raw.
- **Good for:** keeping client trust and staying within contracts.
- **Learn:**
  - A local redaction step swaps names, phone numbers, patient details and amounts for tags before anything leaves the machine.
  - The verifier sees only the task, the output and the rubric.
  - The "replies in my voice" task stays fully local, and you grade it yourself.

### 7. Basic Python and git (just enough)
- **Why:** the system is plain scripts and folders. Git turns undo into one command.
- **Learn:**
  - Running a script.
  - Editing a `.jsonl` file (one case per line).
  - `git log` and `git revert`. Claude writes the code; you only need to read it.

---

## Part B — Tools we'll use, and ones we deliberately won't

| Tool | Use? | Why / why not |
|---|---|---|
| **Ollama** (generator runtime) | ✅ v1 | Already installed and models are on D:. Simple local API with a JSON-schema output mode. The 11434 failure is almost certainly that the server wasn't running. |
| llama.cpp server | ❌ v1 | More control, but more hand-tuning. Only if Ollama limits us. |
| **Qwen3.5-4B Q4_K_M** | ✅ daily | Fits in about 3 GB with an 8K context. |
| Qwen3.5-9B | 🌙 only | 6.6 GB spills into RAM at about 3–6 tok/s. Overnight batches only. |
| LFM3-1.7B | ❌ | Too small for client drafts. |
| **Claude API, Haiku class** (verifier) | ✅ | Cheap grading. About $2–3 a month at 10 grades a day. |
| Claude, mid-tier (coach) | ✅ weekly | One proposal a week, under 10¢. |
| GEPA / DSPy | ❌ v1 | Powerful auto-optimizer, but a black box and needs many cases. Revisit after v1 works. |
| Hermes Agent / Letta / EvoAgentX | ❌ | They grade themselves or are too big to understand. We copy ideas, not code. |
| QLoRA fine-tune (Unsloth) | ❌ v1 | Possible on 6 GB for 1.7–4B models, but slow on Windows. 300 examples teach style, not facts. v2 at the earliest. |
| Auto-rollback machinery | ❌ v1 | Both experts agree: approve by hand first. Git makes manual undo one command. |

---

## Part C — Claude Code skills you already have, and when to call them

| Skill | When | Not for |
|---|---|---|
| `plan-feature` | Turn this into PLAN.md with a dry run (done next) | Coding |
| `build-step` | "do step 3": builds exactly one step, tests it, stops | Big jumps |
| `claude-api` | Before writing the verifier and coach code (right model IDs, prices, caching) | The Ollama side |
| `debug-smart` | When a run breaks: diagnose before retrying | Design questions |
| `handoff` | Save progress when a session gets long | Mid-step |
| `ship-check` | When v1 claims "done" | Early steps |
| `skill-creator` | ❌ Not needed. It's for Claude Code skills; our generator "skills" are plain Markdown files. | — |

---

## What the experts disagreed on, and my pick
- **First task.** The product reviewer says start with client report drafts because that's the money path. The eval expert says start with the easiest task to check.
  - **Pick:** start with *plan my day* to prove the loop cheaply and privately, then *client drafts* second (moved ahead of research).
- **Held-out sets.** The product reviewer says skip them; the eval expert says they're essential.
  - **Pick:** keep them small (8 train / 4 held-out), because without them the score can lie.
- **Coach speed.** **Pick:** one proposal a week, reviewed on one Friday screen, so you don't burn out.

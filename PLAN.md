# Plan: harness v1 (universal skill engine + the infrastructure around it)

Source: IDEA.md, REQUIREMENTS.md, SECURITY.md. Rule: one step per session (`build-step`); each step ends with a check you can see.
**Gate:** only Step 0 starts now. Steps 1–6 start after 3 paying customers (at least 1 repeat) OR 3+ hours a week spent on hand delivery. Steps 7–12 wait for ~30 delivered orders (REQUIREMENTS §4).
**Scope:** v1 automates ONE skill: month-end bank entries (`bank-entries`). A second card exists only to prove the engine runs any card (REQUIREMENTS §5).
**Why not a converter:** PDF→Excel/Tally tools already sell at ₹950–2,000/month and TallyPrime imports PDF statements itself. We sell the work around it: the client's own ledger names, party names from narrations, GST/TDS flags, a balance check, an unknowns list, human review, 24 h turnaround.

## Layout (target)
```
agent.py              CLI: run · review · learn · score  (+ doctor, weekly)
config.toml           models, caps, paths (no secrets)
harness/core.py       config, card loader, LLM client (Ollama + Claude), call cap, log
harness/guard.py      file allow-list, values-only read, reversible redaction, formula guard, coach-edit scan
engine/               gen.py check.py judge.py score.py coach.py learn.py  (the 6 engine files)
skills/<name>/        card.md · rules.md · key.jsonl
tests/                pytest + fake LLM (offline, no cost)
data/                 inputs/ runs/ labels/ fixes/ log.jsonl  (gitignored, on the encrypted volume)
money.csv             (gitignored)
```

## Steps
- [ ] 0. Sell by hand (no code). Buyer: small CA offices and freelance bookkeepers. Offer: "send an unlocked statement + your Tally ledger list, get a reconciled, Tally-import Excel in 24 h". Sample: one made-up statement run through `prompts/statement-to-excel.md`. Channel: LinkedIn + local CA WhatsApp / ICAI branch groups, 3 hand-written messages a day, never automated. Price tests (guesses): ₹400 per statement one-off, or ₹3,000/month for up to 10 statements; UPI. First statement free only with the client's written OK, or on the sample. Each fix you make goes into `skills/bank-entries/rules.md` by hand. Log every message in money.csv. — done when: the gate above is met. Kill rule: 14 days or ~30 messages with no paid order → change the buyer (e.g. small traders directly); 2 buyer changes with nothing → stop. — files: money.csv, prompts/, skills/bank-entries/rules.md
- [ ] 1. Machine setup (after the gate). Python 3.11 venv. Ollama on 127.0.0.1 with qwen3.5:4b (num_ctx 8192, thinking off, temperature 0, fixed seed). gitleaks as a plain git pre-commit hook. Windows 11 Home has no BitLocker: turn on Device Encryption if Settings offers it, else make a VeraCrypt container and mount it as data/. — done when: a test prompt answers in under 30 s, VRAM stays under 5 GB, the same prompt twice gives the same answer, gitleaks blocks a commit with a fake key, and data/ sits on an encrypted volume. — files: scripts/pre-commit, SETUP.md
- [ ] 2. Skill cards (no code). The winner skill plus one contrast card. card.md fields: task, input types, output JSON schema, mode (`direct` = model writes the output; `ops` = model writes table operations), model, checks list, redact on/off. All 12 cases per card (8 train, 4 held-out), each with an expected output. — done when: both cards fill the same template with no special fields. — files: skills/*/
- [ ] 3. Skeleton + tests. Create the layout above, config.toml, requirements.txt pinned with hashes, and pytest with a fake LLM. Add `agent.py doctor`, which checks: Ollama up, key present, gitleaks hook installed, data/ encrypted and mounted, and lists files in data/ older than 30 days (deletes only after you type yes). — done when: `pytest` is green and doctor prints OK/FAIL per item. — files: agent.py, config.toml, requirements*.txt, tests/conftest.py
- [ ] 4. core.py. Load and validate cards. One `call(model, prompt, schema)` covers Ollama and Claude, with temperature 0, a timeout, one retry, JSON-schema output, a cap of 20 Claude calls a day, and a pause on Claude calls after 7 days with no review. Logs go to data/log.jsonl with file hash, model, tokens, cost and score, never text. The key comes from keyring. — done when: tests pass on the fake LLM, one live Ollama call and one live Claude call are logged, and a 21st Claude call is refused. — files: harness/core.py, tests/test_core.py
- [ ] 5. guard.py. Allow-list: .xlsx .csv .txt .docx .pdf, up to 20 MB. Reject .xlsm, .docm and archives. Read xlsx as values only. Before anything goes to Claude, swap names, phones, emails and IDs for placeholders ([NAME_1]) and keep the map locally; put real values back into the output after. The map is never logged or sent. Wrap input in `<data>` tags. Prefix `'` to cells starting with `= + - @`. Scan coach edits: block URLs, emails, "ignore previous", "send", paths, or more than 1,500 tokens. — done when: a test passes for each bad case, and a redact → restore round trip returns the original. — files: harness/guard.py, tests/test_guard.py, tests/files/
- [ ] 6. gen.py + `run`. `agent.py run <skill> <file>` runs guard → card + rules → model → schema check (one retry) → data/runs/<date>/<id>.json plus the delivery file. In `ops` mode the model sees the headers plus 20 sample rows and returns operations from a fixed list (rename, trim, case, date_format, dedupe, split, fill, drop_empty); a generic applier runs them on every row, so sheet size is not limited by context and cell values are never invented. — done when: 3 valid runs in a row per card, one of them a 5,000-row sheet in ops mode, and a test confirms no skill name appears in code. — files: engine/gen.py, agent.py
- [ ] 7. check.py + `review`. Yes/no checks come from the card. `review` shows the newest run; one key (g/f/b) saves a label to data/labels/. Pressing f saves your fix to data/fixes/. — done when: 20 labels are saved (F3 needs 20). — files: engine/check.py, agent.py
- [ ] 8. judge.py. Haiku with a binary rubric. It sees the task, the output, the expected answer from key.jsonl and the rubric, nothing else. Results are cached by output hash. — done when: agreement with your 20 labels is printed and is 85% or more. — files: engine/judge.py
- [ ] 9. Local or Claude? Run the 8 train cases once on Qwen; check.py plus the judge grade them (8 Claude calls at most). — done when: 60% or more → card says `model: local`; less → `model: claude`. — files: skills/*/card.md
- [ ] 10. score.py + `score`. Train cases: check.py only (free). Held-out cases: check.py + judge; the score is the % of rubric items passed, which is finer than pass/fail on 4 cases. That's 4 Claude calls per score, cached. Results go to scores.csv. `--regression` runs check.py only, on the failure pile. — done when: there is a baseline line per card, and scoring twice gives the same number. — files: engine/score.py
- [ ] 11. learn.py + `learn` (F5). `learn <run-id>` turns your fix from data/fixes/ into a new train case (redacted, kept out of the public key) and queues the coach. — done when: one fix becomes a case and `score` counts it. — files: engine/learn.py
- [ ] 12. coach.py + `weekly`. The coach reads train failures, your fixes (redacted) and money.csv, then proposes ONE rules.md diff → guard scan → score → `approve` (git commit) or `reject` (draft deleted). Approve only if held-out doesn't drop and regression passes. `agent.py weekly` prints one line per skill and the 4-box security check. — done when: one approve cycle and one reject cycle work, and a planted bad rule is caught and reverted. — files: engine/coach.py, agent.py

## Edge cases
- Ollama is down → stop with "start Ollama". Never switch to Claude silently.
- No key, or the cap is reached → local work continues; Claude steps stop and say when calls reset.
- Encrypted volume not mounted → `run` refuses, and doctor shows FAIL.
- Wrong type, over 20 MB, macro, empty or unreadable file → rejected with a reason, and nothing is written.
- Output fails the schema twice, or an ops list has an unknown op → rejected, logged, and added to the regression pile.
- A placeholder is left in the output after restore → rejected; it is never delivered.
- `direct` input bigger than the 8k context → rejected with its size. Use an `ops` card, or chunking later.
- Password-protected PDF → refused; the client unlocks it before sending. Never ask for or store the password.
- Scanned (image) PDF → flagged as OCR risk; quote longer turnaround or decline.
- Balance check fails (opening + credits − debits ≠ closing) → not delivered until the gap is explained.
- Bank statement = high-sensitivity financial data → consent line in the offer, redact before Claude, keep on the encrypted volume, delete 30 days after delivery.
- Same file run twice (same hash) → warn and show the earlier run.
- Crash mid-run → output goes to a temp file and is then renamed, so runs/ never holds half an output.
- A coach edit fails the scan or lowers the held-out score → auto-rejected, with the reason shown.

## Assumptions to verify (before Step 3)
- [ ] qwen3.5:4b fits in 6 GB at num_ctx 8192 — check: `ollama run`, then `nvidia-smi`.
- [ ] Ollama honours `format: <JSON schema>`, `think: false` and temperature 0 + seed gives the same output twice — check: two curl calls.
- [ ] Device Encryption is offered on this laptop — check: Settings › Privacy & security. If not, use VeraCrypt.
- [ ] keyring reads and writes Windows Credential Manager — check: a 3-line script.
- [ ] openpyxl `data_only=True` returns values. It returns None for formulas Excel never calculated — check: open one test file.
- [ ] `pip-compile --generate-hashes` and `pip install --require-hashes` work on this Windows Python — check: install one package.

## Decisions
- Engine = 6 files. Shared plumbing lives in 2 harness files plus agent.py, so no engine file repeats LLM or security code.
- Ollama is called over stdlib HTTP. Dependencies: anthropic, keyring, jsonschema, openpyxl, python-docx, pypdf, pytest (+ pip-tools for dev).
- Temperature 0 with a fixed seed: a score is repeatable, so `git revert` really restores it (F6). No majority vote needed.
- `ops` mode for tables: the 4B model plans, code applies, so large sheets fit and no values are invented.
- Redaction is reversible, so Claude-made outputs come back with real names.
- The judge sees the expected answer and runs only in `score` and Step 9: at most 8 Claude calls on any one day, under the cap of 20.
- `learn` = correction → case (F5). Drafting new cards from examples moves to Later.
- Audio is dropped from v1 (no transcriber). Fixes go to gitignored data/fixes/, never to the public key.jsonl.

## Done for v1 (matches REQUIREMENTS §6)
One skill passes 8 of 10 of your reviews on real orders. Its held-out score rises above baseline after at least 1 approved coach edit. A planted bad rule is caught and reverted. Cost stays under ₹1,000 a month. money.csv has at least 10 real offers. The weekly security check passes.

## Later
Card drafting from examples (`learn --new`), chunking for big `direct` inputs, a second automated skill, batch mode.

## Notes for next session
Refocused on one paid service (bank entries for CAs) after checking competitors. Next: Step 0 only. Use prompts/statement-to-excel.md for every hand delivery.

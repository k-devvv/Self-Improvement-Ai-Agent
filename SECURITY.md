# SECURITY.md: how the agent protects you and your clients

Threat model: one person, one Windows laptop, a public repo, and client files from strangers. The main risks are a leaked API key, leaked client data, a malicious input file, and the self-improvement loop being tricked into changing its own rules. The layers below are ordered by importance.

## 1. Secrets
- The Anthropic key lives in Windows Credential Manager (Python `keyring`). It is never in code, `.env` or git.
- The key belongs to its own Anthropic workspace, with a monthly spend limit set in the console.
- `gitleaks` runs as a pre-commit hook and blocks any commit that contains a key or password.
- If the key leaks: revoke it in the console, make a new one, and note it in `DECISIONS.md`.

## 2. Client data (this repo is PUBLIC)
- `data/` (inputs, runs, labels, fixes, log) and `money.csv` are in `.gitignore`. Client files never touch git.
- Answer-key cases use made-up or fully anonymized data only.
- Client files sit under `data/` on an encrypted volume: Device Encryption if Settings offers it, else a VeraCrypt container mounted as `data/` (Windows 11 Home has no BitLocker). They are deleted 30 days after delivery, after you confirm.
- Names, phone numbers, emails and IDs are redacted locally before any text goes to Claude. The local model sees raw data; Claude sees redacted data.
- Logs record file names, token counts and scores. They never record client text.

## 3. Untrusted input files
- Every client file is data, never instructions. Prompts wrap the content in tags and say so.
- Allowed types: .xlsx .csv .txt .docx .pdf (no audio in v1). Max 20 MB. Archives, executables and macro files (.xlsm, .docm) are rejected.
- Spreadsheets are read with formulas off (values only). Macros are never run.
- CSV/formula injection guard: any output cell starting with `= + - @` gets a leading `'` before delivery.
- Windows Defender scans each file when it is saved to `data/inputs/`.

## 4. The agent can't act
- The agent can only read input files and write to `data/runs/`. It has no tools to send, post, delete, browse or pay.
- Network: Ollama on `127.0.0.1` only (never `0.0.0.0`), plus `api.anthropic.com`. Nothing else.
- Output must match the card's JSON schema. Anything else is rejected.

## 5. Self-improvement guardrails
- The coach can only propose edits to one skill's `rules.md`. It never touches code, `card.md`, keys or other skills.
- Every proposed edit is scanned before you see it. It is blocked if it contains URLs, emails, "ignore previous", "send", or file paths, or if it is over 1,500 tokens.
- Nothing changes without your `approve`. Every change is a git commit, and `git revert` undoes it.
- A change is accepted only if the held-out score doesn't drop and regression passes, so a poisoned rule that hurts quality is caught.

## 6. Supply chain
- Few dependencies, pinned with hashes (`pip install --require-hashes`), with `pip-audit` once a month.
- Models are pulled only from the official Ollama library.

## 7. Accounts
- 2FA (an authenticator app, not SMS) on email, GitHub, Anthropic and every selling platform.
- Commits use the GitHub noreply email, not your personal address.
- One account per platform. AI help is disclosed and a human reviews every order.

## 8. Weekly check (5 minutes, part of the weekly review)
- [ ] Spend this week is under the cap
- [ ] No files in `data/` older than 30 days
- [ ] `git log` shows only changes you approved
- [ ] `gitleaks detect` is clean

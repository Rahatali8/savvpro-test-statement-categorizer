# AI usage

> DRAFT: the facts below come from the build session. Rewrite it in your own words, fill in
> the TODO token numbers and add the screenshots before you submit.

## Tools

| Tool | Used for |
| --- | --- |
| Claude Code (Claude Opus 5.5) | Planning from my project plan, writing backend code, tests, Docker/CI and docs drafts |
| Claude Code subagent (general-purpose) | Built the Next.js frontend **in parallel** with the backend, working from `docs/api-contract.md` |
| `taste-impeccable-workflow` skill | Frontend design: stage 1 (taste-skill) set the direction and made the first build; stage 2 (Impeccable) detected, critiqued, audited, fixed and polished it |
| Project subagent definitions | `.claude/agents/test-writer.md`, `.claude/agents/reviewer.md` |

**No AI is used inside the app.** The categorizer is plain Python rules. CI fails if
`openai|anthropic|gemini` appears anywhere in `backend/app/`.

## How the work was split

1. **Plan → rules.** I wrote `Statement Categorizer — Project Plan.md` (in Roman Urdu) and
   gave it to Claude Code. It wrote `CLAUDE.md` (hard rules: no LLM in the brain, fake
   data only, branch `candidate-Rahatali8` only, the human commits) and the folder scaffold.
2. **Contract first.** Before any parallel work it wrote `docs/api-contract.md`, so the
   frontend and backend had one agreed source of truth.
3. **Parallel.** A frontend subagent built `/frontend` while the main session built
   `/backend`. Each owned its own folder, and neither committed.
4. **Backend phases:** fake sample CSVs → parser + tests → brain + tests → API + DB + tests
   → Docker + CI → docs.
5. **I reviewed, ran and committed** every phase myself.

## Where the AI got things wrong (and how it was caught)

| # | What went wrong | How it was caught | Fix |
| --- | --- | --- | --- |
| 1 | `dateutil` with `dayfirst=True` silently read ISO `2026-05-03` as **5 March** | Smoke-running the parser on `messy.csv` showed row 6 as `2026-03-05` | ISO dates are parsed explicitly before dateutil; covered by `test_iso_dates_are_never_flipped_by_day_first` |
| 2 | "Not spending" total **netted** transfers out against card payments in (−500 + 800 = 300 instead of 1300) | `test_summary_totals` failed | Not-spending and review totals now add up absolute amounts |
| 3 | `datetime.utcnow()` crashed every DB write; the installed SQLModel version requires timezone-aware datetimes | 13 API tests failed on the first run | Switched to `datetime.now(UTC)` |
| 4 | The first rules made `JOES CAFE` → Meals (keyword `CAFE`, 75%), but the task says this exact merchant must go to Needs Review | Comparing the first classifier output against TASK.md | `CAFE`/`COFFEE` became *weak* keywords (50%), so the row goes to review with a "Meals" hint |
| 5 | A test expected `BLUE DOOR STUDIO 22` to be the same merchant as `BLUE DOOR STUDIO`, but the normalizer keeps short numbers on purpose (`MICROSOFT 365`) | Failing test | The test was wrong, not the code; fixed the test input |
| 6 | The plan put learned rules *after* special rules, so a user's fix of a "transfer" row would never stick | Design review while writing the brain | Learned rules (exact merchant) now run first; documented in DESIGN.md |
| TODO | Add anything else you noticed while reviewing or clicking through the UI | | |

## Tokens

| Session | What | Tokens / cost |
| --- | --- | --- |
| Main session | Plan → backend, tests, Docker, CI, docs | TODO from `/cost` or the usage screen |
| Frontend subagent | Next.js UI with the taste → Impeccable workflow | TODO (counted in the session total) |
| **Total** | | **TODO** |

Screenshots are in [`docs/ai-usage/`](docs/ai-usage/). TODO: add `/cost` or usage-screen
screenshots for each session.

## What I'd do differently

TODO: your own reflection, for example where giving Claude smaller tasks would have saved
time, or what you would check earlier.

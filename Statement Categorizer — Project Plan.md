# Statement Categorizer — Project Plan

Oct 26, 2026 · @rahat

## Khulasa

Hum **"LedgerLite"** naam ki ek web app banayenge. Ek freelancer is mein apne business credit card ki statement (CSV) upload karega, aur app har transaction ki category khud batayegi. Jahan app ko yaqeen nahi hoga, wahan woh guess nahi karegi balke row ko **Needs Review** mein daal degi.

- **User:** "Sara Khan", ek fake freelance designer jise tax ke liye apne kharche alag alag categories mein chahiye.
- **Final result:** upload karo, categories dekho, shaky rows theek karo, har category ka total dekho, aur CSV export karo.
- **Sab se bara rule:** category decide karne wala "brain" hamara apna Python code hoga. Koi GPT, Claude ya Gemini API app ke andar use nahi hogi.
- **Code:** Claude Code se likhwayenge, aur har step ka hisaab `AI_USAGE.md` mein rakhenge.

**Deadline:** aaj, Monday 26 October, raat **12:00 AM (PKT)**. Sirf yahi ek test karna hai, aur late submission accept nahi hogi.

## Meri recommendations

Sab se ahem mashwara yeh hai ke **feature kam rakho lekin har feature mazboot aur tested ho.** Evaluator ko 5 mukammal cheezein 15 adhoori cheezon se zyada achi lagengi.

1. **Sirf CSV support karo, PDF nahi.** PDF parsing mein waqt zaya hoga. DESIGN.md mein likh do ke PDF aage ka kaam hai.
2. **Brain ko alag "pure" Python module rakho** (`backend/app/engine/`). Is mein koi FastAPI ya database code na ho, taake tests asaan rahein aur evaluator ko saaf dikhe ke brain aap ka apna hai.
3. **Har faisle ke saath "kyun" bhi dikhao,** jaise "ADOBE keyword match hua (Software, 95%)". Isay explainability kehte hain, aur yeh bohat strong point hai.
4. **Rules ko code mein hardcode mat karo.** Unhein `rules.yaml` file mein rakho, taake naye rules bina code badle add ho sakein. Production wali soch yahi hai.
5. **User correction ko yaad rakhna (learning)** zaroor banao. Yeh chhota sa feature hai magar bohat impressive lagta hai.
6. **Pehle din se tests likho,** aakhir mein nahi. Har messy case ka kam az kam ek test ho.
7. **Claude Code ko ek waqt mein ek chhota kaam do,** poora project ek saath nahi. Har phase ke baad khud chala kar check karo aur commit karo.
8. **AI\_USAGE.md ka hisaab saath saath rakho.** Har session ke baad `/cost` ya usage screen ka screenshot `docs/ai-usage/` folder mein save karo, aur jo ghalti Claude Code kare woh note karo.
9. **Chhote chhote commits karo,** misaal ke taur par `feat: add csv parser`. Evaluator commit history bhi dekhte hain.
10. **Bonus ke liye:** Docker Compose (ek command se chale) aur GitHub Actions CI (har push par tests chalein).

## Features

Pehle saare **Must-have** features banao. Waqt bache to **Nice-to-have** add karo.

| # | Feature | Kya karega | Priority |
| --- | --- | --- | --- |
| 1 | CSV upload | Drag-and-drop ya button se file upload. Sirf `.csv`, max 5 MB | Must-have |
| 2 | Smart CSV reader | Alag column names pehchane (Date/Txn Date, Amount ya Debit+Credit), alag date formats samjhe | Must-have |
| 3 | Rule-based brain | Har row ki category, confidence % aur "kyun" ka reason | Must-have |
| 4 | Needs Review | Kam yaqeen wali rows alag highlight hon | Must-have |
| 5 | Fix category | Dropdown se category badlo, foran save ho | Must-have |
| 6 | Category totals | Har category ka total, aur "Not spending" (transfer, payment) alag | Must-have |
| 7 | Special cases | Refund, transfer, card payment, fees, duplicates sahi handle hon | Must-have |
| 8 | Error messages | Ghalat file ya khali file par saaf message, crash nahi | Must-have |
| 9 | Tests | Brain, parser aur API ke pytest tests | Must-have |
| 10 | Learning from fixes | User ka fix yaad rahe, agli dafa usi merchant par khud lage | Nice-to-have (zaroor karo) |
| 11 | Export CSV | Final categorized statement download | Nice-to-have |
| 12 | Filters | All / Needs Review / category ke hisaab se table filter | Nice-to-have |
| 13 | Sample file button | "Try with sample" se fake statement load ho, evaluator ke liye asaan | Nice-to-have |
| 14 | Docker + CI | `docker compose up` se chale, GitHub Actions par tests | Nice-to-have |

**Categories (fixed list):** Software & Subscriptions, Travel, Meals, Office Supplies, Internet & Phone, Advertising, Bank Fees, Income, Refund, Transfer (not spending), Card Payment (not spending), Personal, aur Needs Review.

## Brain kaise kaam karega

Brain har row ko 4 steps se guzarta hai. Category sirf tab lagti hai jab confidence 0.70 ya us se zyada ho aur sirf ek category jeete. Warna row **Needs Review** mein jati hai.

&#91;embedded content: brain ka flow · 4 steps, 1 faisla, 1 learning loop\]

User jab Needs Review wali row theek karta hai, to woh merchant "learned rule" ban jata hai aur agli dafa step 3 par khud pehchana jata hai.

**Confidence kaise banta hai:**

| Match ki qisam | Confidence | Misaal |
| --- | --- | --- |
| Learned rule (user ka fix) | 1.00 | `JOES CAFE` → Meals |
| Special rule | 0.95 | `AMAZON REFUND` → Refund |
| Exact merchant (rules.yaml) | 0.90 | `ADOBE` → Software |
| Keyword match | 0.75 | `...AIRLINES...` → Travel |
| Kamzor keyword | 0.50 → Needs Review | `STORE` |
| Do categories barabar | Needs Review | `AMAZON` (office ya personal?) |
| Koi match nahi | Needs Review | `JOES CAFE 9921` |

**Har row ka result 4 cheezein deta hai:** category, confidence, reason (jaise "Keyword ADOBE matched Software") aur source (rule / learned / user). Duplicate check alag se chalta hai: same date, same amount aur same merchant ho to doosri row flag hoti hai.

## Messy cases aur unka hal

Task kehta hai ke yeh cases dhoondna aap ka kaam hai. Har case ke liye ek fake CSV row aur ek test zaroor ho.

| Case | Misaal | App kya karegi |
| --- | --- | --- |
| Refund | `AMAZON REFUND +30.00` | **Refund** category. Kharche mein se minus, income nahi |
| Apna transfer | `TRANSFER TO SAVINGS -500` | **Transfer (not spending)**, totals mein shamil nahi |
| Card bill payment | `PAYMENT THANK YOU +800` | **Card Payment (not spending)** |
| Anjaan merchant | `JOES CAFE 9921` | **Needs Review**, guess nahi |
| Do categories takrayein | `AMAZON` (office ya personal?) | Needs Review, reason: "multiple matches" |
| Positive amount, refund lafz nahi | `+15.00 UNKNOWN CREDIT` | Needs Review. Income ya refund ka guess nahi |
| Duplicate row | Same date, description, amount do dafa | Doosri row par "Possible duplicate" flag + Needs Review |
| Alag date formats | `03/05/2026`, `2026-05-03`, `5 Mar 2026` | Sab ek format mein convert. Ambiguous ho to warning |
| Debit/Credit alag columns | `Debit=54.99, Credit=` | Ek signed amount bana do |
| Amount mein comma ya currency | `"1,250.00"`, `$54.99`, `(54.99)` | Saaf number. Bracket wala negative |
| Khali ya kharab row | Amount missing | Row skip, user ko warning: "Row 14 skipped" |
| Bank fees | `ANNUAL FEE`, `LATE FEE`, `FX FEE` | **Bank Fees** |
| Foreign currency | `FIGMA USD 15.00` | Category normal, note mein currency |
| Ajeeb description | `SQ *JOES CAFE`, `PAYPAL *ADOBE` | Prefix (`SQ *`, `PAYPAL *`) hata kar asal merchant dhoondo |
| Khali file / ghalat file | `.pdf` ya 0 rows | Saaf error message, crash nahi |

## UI plan (AIT LMS jaisa clean admin style)

UI bilkul aap ke LMS screenshot jaisa hoga: safed cards, halka grey background, ek teal accent, saaf table aur chhote status pills. **Koi emoji nahi, koi gradient nahi, koi bhari shadow nahi.** Frontend **Next.js** par banega.

**Design rules (Claude Code ko yahi dene hain):**

| Cheez | Rule |
| --- | --- |
| Background | Page halka grey-white (`#F7F9F9`), cards safed |
| Accent color | Sirf ek teal/green (`#1F7A68` ke qareeb). Primary button, active tab, icons |
| Borders | Patli `1px` border (`#E5EAE9`), corners `rounded-xl`, shadow bilkul halki ya nahi |
| Font | Ek saaf sans font (Inter ya Lexend). Headings semibold, labels chhote |
| Icons | `lucide-react` outline icons, soft tinted circle ke andar |
| Status pills | Chhote rounded pills: Auto (teal), Needs Review (amber), Learned (blue), Duplicate (red), Not spending (grey) |
| Numbers | Amounts right-aligned, `tabular-nums`. Kharcha normal, paisa wapas aaya to hara |
| Mana hai | Emoji, gradients, glassmorphism, neon colors, bari animations |

**Layout (LMS jaisa):**

1. **Left sidebar:** upar logo "LedgerLite" aur neeche "Tax Categorizer". Nav: Dashboard, Statements, Needs Review (count badge), Learned Rules. Neeche "Sign out".
2. **Top bar:** breadcrumb (Home > Statements), aur right par fake user avatar "SK, Sara Khan, Freelancer".
3. **Page header:** icon box + title "Statement review" + ek line subtitle. Right par teal button "Upload statement".
4. **4 stat cards:** uppercase chhota label, bara number, right par icon circle. Cards: Transactions · Auto-categorized · Needs Review · Total spending.
5. **Filter card:** "Filter transactions" heading, phir 3 fields ek line mein: Search (merchant) · Category · Status.
6. **Tabs (count ke saath):** All · Needs Review · Duplicates · Not spending, LMS ki tarah grey pill counts.
7. **Table:** headers uppercase grey. Columns: Date · Merchant (bold naam, neeche chhota grey original description) · Category (dropdown) · Amount · Confidence · Status pill. Needs Review rows ka halka amber left border.
8. **Category totals card:** har category ka naam, total aur ek patli horizontal bar. Neeche "Export CSV" button.
9. **Empty state:** pehli dafa upload ka box, "Drop CSV here" aur "Try with sample statement" link.

**Screenshot ke baare mein zaroori baat:** aap ka LMS screenshot asli data hai (aap ka naam, asli courses). Task kehta hai **sirf fake data**, is liye yeh screenshot **repo mein commit mat karna.** Claude Code ko sirf chat mein drag karke dikhao, ya repo ke bahar kisi folder mein rakho.

**Design workflow:** Stage 1 mein taste-skill screenshot dekh kar pehla build banaye. Stage 2 mein Impeccable detect, critique, audit, fix aur polish kare. Agar Claude Code mein `/impeccable` nahi chalta, to pehle skill wahan install karo. AI\_USAGE.md mein is skill ka zikr zaroor karna.

## Tech stack aur structure

**Stack:**

| Hissa | Cheez | Kyun |
| --- | --- | --- |
| Backend | Python 3.11 + FastAPI | Tez, simple, auto API docs (`/docs`) |
| CSV padhna | Python `csv` + `pydantic` | Halka aur validation built-in |
| Dates | `python-dateutil` | Alag date formats samajhta hai |
| Rules | `rules.yaml` + `PyYAML` | Rules code se bahar |
| Database | SQLite + SQLModel | Koi setup nahi, learned rules aur statements save |
| Tests | `pytest` + `httpx` | Brain aur API dono test |
| Code quality | `ruff` | Lint + format |
| Frontend | Next.js (App Router) + TypeScript | Task mein allowed, routing built-in |
| Styling | Tailwind CSS | Clean UI jaldi |
| Run | Docker Compose | Ek command se poori app |
| CI | GitHub Actions | Har push par tests + lint |

**Folder structure:**

```
savvpro-test-statement-categorizer/
├── backend/
│   ├── app/
│   │   ├── main.py            # FastAPI app
│   │   ├── api/routes.py      # endpoints
│   │   ├── engine/            # BRAIN (pure Python)
│   │   │   ├── normalizer.py
│   │   │   ├── rules.py
│   │   │   ├── classifier.py
│   │   │   └── rules.yaml
│   │   ├── parsing/csv_parser.py
│   │   ├── models.py          # pydantic/SQLModel
│   │   └── db.py
│   ├── tests/
│   ├── pyproject.toml
│   └── Dockerfile
├── frontend/
│   ├── app/ + components/ + lib/api.ts (Next.js)
│   └── Dockerfile
├── sample_data/               # sirf fake CSVs
├── docs/ai-usage/             # token screenshots
├── .github/workflows/ci.yml
├── docker-compose.yml
├── .env.example
├── README.md
├── DESIGN.md
└── AI_USAGE.md
```

**API endpoints:**

| Method | Path | Kaam |
| --- | --- | --- |
| POST | `/api/statements` | CSV upload, categorized rows wapas |
| GET | `/api/statements/{id}` | Ek statement ki saari rows |
| PATCH | `/api/transactions/{id}` | User category badle, rule bhi seekhe |
| GET | `/api/statements/{id}/summary` | Category totals |
| GET | `/api/statements/{id}/export` | CSV download |
| GET | `/api/health` | App chal rahi hai ya nahi |

**Har transaction ka data:** `id, date, description, normalized_merchant, amount, currency, category, confidence, reason, needs_review, is_duplicate, source` (`source` = rule / learned / user).

## Tests ka plan

Target yeh hai ke brain aur parser ki coverage 80% se upar ho. Command: `pytest --cov=app`.

**Brain tests (`test_classifier.py`)**

- `ADOBE CREATIVE CLOUD` → Software, confidence > 0.8
- `UBER *TRIP 8812` → Travel (normalization ke baad)
- `AMAZON REFUND +30` → Refund, Income nahi
- `TRANSFER TO SAVINGS` → Transfer, totals mein nahi
- `JOES CAFE 9921` → Needs Review
- Do categories match hon → Needs Review, reason "multiple matches"
- Positive anjaan amount → Needs Review

**Parser tests (`test_csv_parser.py`)**

- Alag column names wali 3 sample CSVs sahi parse hon
- Debit/Credit alag columns → sahi signed amount
- `"1,250.00"`, `$54.99`, `(54.99)` → sahi number
- 3 date formats → same date
- Kharab row skip ho aur warning aaye
- Khali file → saaf error

**Duplicate aur learning tests**

- Same row do dafa → doosri flag ho
- User ne JOES CAFE ko Meals kiya → agli upload mein khud Meals (source = learned)

**API tests (`test_api.py`)**

- Upload → 200 aur rows wapas
- `.pdf` upload → 400 error
- PATCH category → save ho
- Summary totals sahi hon (refund minus, transfer bahar)

## Git aur branch rules

**Saara kaam sirf `candidate-Rahatali8` branch par hoga.** `main` par kabhi commit nahi karna, aur koi doosri branch bhi nahi banani.

1. **Kaam shuru karne se pehle** har dafa check karo:

```bash
git branch --show-current   # output: candidate-Rahatali8
```

2. **Galti se `main` par ho** to pehle switch karo: `git checkout candidate-Rahatali8`
3. **Git worktrees use nahi karne.** Woh alag branch bana dete hain, aur aakhir mein merge ka jhanjhat ho jata hai. Agent team aur subagents isi ek branch aur isi folder mein kaam karenge.
4. **Commit aur push sirf aap karoge,** ek hi terminal se. Agents se commit nahi karwana, taake history saaf rahe.
5. **Push hamesha isi branch par:** `git push origin candidate-Rahatali8`
6. **PR:** apne fork ki `candidate-Rahatali8` branch se `savvpro/savvpro-test-statement-categorizer` ki `main` par.
7. **Yeh rule CLAUDE.md mein bhi likha hoga** (Phase 0 ka prompt yeh add karwata hai), taake har agent ko pata ho.

## Multi-agent setup

Multi-agent sirf wahan use hoga jahan kaam waqai parallel ho sakta hai. Is ka sab se bara faida yeh hai ke frontend **\~3:30 PM se backend ke saath saath** banna shuru ho jayega, aur 2-3 ghante bachenge.

| Phase | Tareeqa | Kyun |
| --- | --- | --- |
| 0-1 Setup, fake data | Ek session | Chhota kaam hai |
| 2-3 Parser + Brain | Ek session + subagents (test-writer, reviewer) | Brain parser par depend karta hai |
| 4-5 API + Frontend | Agent team, 3 teammates parallel | Alag folders hain, ek saath ban sakte hain |
| Aakhir mein | Reviewer subagent | LLM call, secrets aur real data ka check |

**1. Do subagents** (`.claude/agents/` mein, Phase 0 ke baad):

`.claude/agents/test-writer.md`

```markdown
---
name: test-writer
description: Writes and runs pytest tests for backend modules, especially messy statement edge cases.
model: sonnet
---
You write pytest tests for /backend. Cover refunds, transfers, card payments, duplicates, date formats, broken rows and Needs Review. Run pytest and report failures. Never edit app code, only tests. Never commit.
```

`.claude/agents/reviewer.md`

```markdown
---
name: reviewer
description: Reviews code for task rule violations before commits.
model: sonnet
---
Check: no LLM API calls (openai, anthropic, gemini) anywhere in app code, no secrets or real data committed, every edge case has a test, current branch is candidate-Rahatali8. Report issues with file:line. Do not edit files.
```

**2. Agent team on karo** (experimental feature). `~/.claude/settings.json` mein:

```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

**3. Phase 3 ke baad (\~3:30 PM) team ka prompt:**

```
First write docs/api-contract.md with every endpoint, request and JSON response shape. Then create an agent team with 3 teammates, all working on the current branch candidate-Rahatali8 in this same folder (no worktrees, no new branches, no commits):
- backend: owns /backend/app only, builds FastAPI endpoints + SQLite per the contract
- frontend: owns /frontend only, builds the Next.js UI per the contract and the design rules in CLAUDE.md, using the taste-impeccable-workflow skill
- qa: owns /backend/tests and /sample_data only, writes API and edge-case tests, runs pytest, reports bugs to backend
No teammate edits files outside its folder. Wait for all teammates before finishing.
```

**Dhyaan rakhne wali baatein:**

- API contract pehle likhwana zaroori hai, warna frontend aur backend alag cheezein bana denge.
- Har agent ka folder alag ho. Do agents ek file edit karein to kaam overwrite ho jata hai.
- Team mein 30 minute tak masla rahe to team band karo aur do terminals mein do sessions chalao, ek `/backend` aur ek `/frontend`. Dono isi branch aur isi folder mein.
- Har teammate alag Claude instance hai, is liye tokens zyada lagenge. Har session ki usage ka screenshot lo.
- AI\_USAGE.md mein yeh orchestration zaroor likho: kaun se agents banaye, kya parallel kiya, kahan masla aaya, aur kitne tokens lage.

## Kaam ke phases (Claude Code ke saath)

Har phase ka tareeqa ek hi hai: prompt do, code padho, khud chala kar check karo, ghalti ho to note karo, commit karo, aur `/cost` ka screenshot lo. Prompts English mein hain kyun ke Claude Code English mein zyada sahi samajhta hai. Inhein seedha copy-paste kar sakte hain.

**Aaj ka time table** (abhi 12:30 PM, deadline raat 12:00 AM, yaani \~11.5 ghante):

| Waqt (PKT) | Phase | Kaam |
| --- | --- | --- |
| 12:30 – 1:00 PM | 0 | Setup, CLAUDE.md, folders |
| 1:00 – 1:30 PM | 1 | Fake CSV files |
| 1:30 – 3:00 PM | 2 | CSV parser + tests |
| 3:00 – 5:30 PM | 3 | Brain + tests (sab se ahem) |
| 5:30 – 7:00 PM | 4 | API + database + tests |
| 7:00 PM |  | **Pehla push + PR khol do** (baad ke push khud PR mein aa jayenge) |
| 7:00 – 9:30 PM | 5 | Next.js UI + Impeccable polish |
| 9:30 – 10:00 PM | 6 | CI (Docker sirf agar waqt ho) |
| 10:00 – 11:00 PM | 7 | README, DESIGN, AI\_USAGE |
| 11:00 – 11:30 PM | 8 | Khud test, aakhri push |
| 11:30 PM – 12:00 AM |  | Buffer. Is waqt naya feature nahi |

**Agar peeche reh jao to yeh cheezein is order mein chhodo:** Docker → Export CSV → Learned Rules page → filters. Brain, tests, Needs Review aur teeno docs **kabhi mat chhodna.**

**Har phase ke baad push karo:** `git add . && git commit -m "feat: ..." && git push origin candidate-Rahatali8`. Agar aakhir mein kuch toot bhi jaye, to pichla kaam PR mein mehfooz rahega.

**Phase 0: Setup (12:30 – 1:00 PM)**

Repo folder mein `claude` chalao. Pehle CLAUDE.md banwao taake Claude Code ko poora project yaad rahe:

```
Read TASK.md. We are building a bank statement categorizer for a freelance designer. Create a CLAUDE.md with these rules: Python 3.11 FastAPI backend in /backend, Next.js (App Router) + TypeScript + Tailwind + lucide-react frontend in /frontend. The categorization brain lives in backend/app/engine and must be pure rule-based Python: NEVER call any LLM API (OpenAI, Anthropic, Gemini) for decisions. Only fake data. No secrets in code. Write pytest tests for every module. Git: work ONLY on branch candidate-Rahatali8; never commit to main, never create other branches or git worktrees, and do not commit or push yourself - the human commits. Then scaffold the folder structure, pyproject.toml (fastapi, uvicorn, pydantic, sqlmodel, python-dateutil, pyyaml, pytest, pytest-cov, httpx, ruff), .gitignore, .env.example. Do not write features yet.
```

**Phase 1: Fake data (1:00 – 1:30 PM)**

```
Create 3 fake CSV statements in /sample_data for a freelance designer named Sara Khan. bank_a.csv: columns Date,Description,Amount (MM/DD/YYYY). bank_b.csv: columns Txn Date,Details,Debit,Credit (YYYY-MM-DD). messy.csv: mixed date formats, "1,250.00" and (54.99) amounts, a duplicate row, an empty amount row, SQ * and PAYPAL * prefixes. Include software, travel, meals, fees, refunds, transfers to savings, card payments, unknown merchants, foreign currency. ~40 rows each. All fake.
```

**Phase 2: CSV parser + tests (1:30 – 3:00 PM)**

```
Build backend/app/parsing/csv_parser.py. It must auto-detect column names (date/description/amount or debit+credit), parse multiple date formats with dateutil, clean amounts (commas, currency symbols, parentheses = negative), skip broken rows and return warnings with row numbers, and raise a clear error for empty/invalid files. Return a list of pydantic RawTransaction objects. Write tests in backend/tests/test_csv_parser.py using the sample files. Run the tests and fix failures.
```

**Phase 3: Brain + tests (3:00 – 5:30 PM, sab se ahem)**

```
Build the rule-based brain in backend/app/engine. normalizer.py: uppercase, strip prefixes like SQ *, PAYPAL *, TST*, remove reference numbers and extra spaces. rules.yaml: categories with keywords, exact merchants and weights. classifier.py: (1) special rules first: refund, transfer, card payment, bank fees; (2) learned user rules; (3) keyword rules; (4) compute confidence 0-1; if confidence < 0.70, or multiple categories tie, or an unknown positive amount -> category 'Needs Review'. Every result returns category, confidence, reason (human readable) and source. Also add duplicate detection (same date+amount+normalized merchant). No LLM. Write thorough tests in test_classifier.py for every edge case in DESIGN notes. Run tests.
```

**Phase 4: API + database (5:30 – 7:00 PM, phir push + PR)**

```
Build FastAPI endpoints: POST /api/statements (upload csv, parse, classify, save in SQLite via SQLModel), GET /api/statements/{id}, PATCH /api/transactions/{id} (update category, set source=user, and save a learned rule for that normalized merchant), GET /api/statements/{id}/summary (totals per category; refunds reduce spending; transfers and card payments shown separately as not spending), GET /api/statements/{id}/export (CSV), GET /api/health. Only accept .csv up to 5MB. Add CORS for the frontend. Write tests in test_api.py with httpx TestClient.
```

**Phase 5: Next.js frontend (7:00 – 9:30 PM)**

LMS ka screenshot Claude Code ki chat mein attach karo (repo mein commit nahi karna). Phir yeh prompt do:

```
Use the taste-impeccable-workflow skill. Build the frontend in /frontend with Next.js (App Router) + TypeScript + Tailwind + lucide-react. Match the visual style of the attached admin dashboard screenshot (do NOT copy its data or commit it): light grey page (#F7F9F9), white cards with 1px #E5EAE9 borders and rounded-xl corners, one teal accent (~#1F7A68), clean sans font (Inter or Lexend), lucide outline icons inside soft tinted circles, small rounded status pills. NO emojis, NO gradients, NO glassmorphism, minimal shadows.
Layout: left sidebar (brand 'LedgerLite' / 'Tax Categorizer'; nav Dashboard, Statements, Needs Review with count badge, Learned Rules; Sign out at bottom); top bar with breadcrumb and fake user 'Sara Khan'; page header with icon box, title 'Statement review', subtitle and teal 'Upload statement' button; 4 stat cards (uppercase label, big number, icon circle): Transactions, Auto-categorized, Needs Review, Total spending; 'Filter transactions' card with Search, Category and Status fields; tabs with count pills: All, Needs Review, Duplicates, Not spending; table with uppercase grey headers: Date, Merchant (bold normalized name + small grey raw description), Category (select), Amount (right aligned, tabular-nums), Confidence, Status pill (Auto teal, Needs Review amber, Learned blue, Duplicate red, Not spending grey); Needs Review rows get a thin amber left border; category totals card with thin bars and Export CSV button; empty state with dropzone and 'Try with sample statement' link.
Changing a category calls PATCH /api/transactions/{id} and refreshes stats and totals. API base URL from NEXT_PUBLIC_API_URL. Handle loading, empty and error states; responsive.
Stage 1 (taste-skill): set direction and build. Stage 2 (Impeccable): detect, critique, audit against the screenshot style, then fix and polish. Report what changed in each stage.
```

**Phase 6: Docker + CI (9:30 – 10:00 PM, Docker optional)**

```
Add Dockerfiles for backend and frontend and a docker-compose.yml so `docker compose up` runs the whole app. Add .github/workflows/ci.yml running ruff and pytest on every push. Make sure no secrets are committed.
```

**Phase 7: Docs (10:00 – 11:00 PM)**

README aur DESIGN Claude Code se likhwao. **AI\_USAGE.md khud likho** (neeche wala section dekho), kyun ke yeh aap ka apna tajurba hai.

**Phase 8: Khud test karo aur submit karo (11:00 – 11:30 PM)**

Teeno sample files upload karo, har messy case dekho, aur ek row fix karke learning check karo. Phir neeche wali checklist poori karo.

## Documents aur submission

**README.md mein kya ho:**

- App kya karti hai (2 lines) aur ek screenshot
- Setup: `docker compose up`, ya backend aur frontend alag alag chalane ke commands
- Tests kaise chalayein: `pytest`
- Sample files kahan hain

**DESIGN.md mein kya ho:**

- Use case: Sara (freelance designer) aur uska masla
- Brain ka flow (neeche wala diagram) aur har step ka kaam
- Confidence kaise nikalta hai aur 0.70 threshold kyun rakha
- Messy cases ki table aur har ek ka hal
- Kya nahi kiya aur kyun (PDF, multi-currency conversion), aur aage kya karte

**AI\_USAGE.md mein kya ho (yeh khud likhna):**

- Tools: Claude Code (aur taste-impeccable-workflow skill), aur jo bhi aur use kiya
- Har phase mein Claude Code se kya karwaya
- **Kahan ghalat hua:** asal misaalein, jaise "refund ko income bana diya, test se pakda" ya "date format ulta parse kiya"
- **Tokens:** har session ka `/cost` total, aur screenshots `docs/ai-usage/` mein link karke

**Submit karne se pehle checklist:**

- [ ] Branch `candidate-Rahatali8` par saara kaam hai
- [ ] `pytest` sab pass, `ruff check` saaf
- [ ] `docker compose up` se app chal jati hai
- [ ] Koi `.env`, API key ya password commit nahi hua (`git log -p | grep -i key` se check)
- [ ] Sirf fake data, koi asli bank statement nahi
- [ ] Code mein kahin OpenAI/Anthropic/Gemini ki API call nahi
- [ ] README, DESIGN, AI\_USAGE teeno mojood
- [ ] Token screenshots `docs/ai-usage/` mein
- [ ] Push: `git push origin candidate-Rahatali8`
- [ ] PR: base `savvpro/savvpro-test-statement-categorizer` (`main`), title `feat: complete statement-categorizer assessment`
- [ ] Deadline se pehle PR khul chuka hai

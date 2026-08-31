# Code Map — where everything is

**For:** walking into the demo not having touched `Src/` in weeks.
**Line numbers verified:** 29 August 2026, commit `24c35d6` (+ docs commits).

If a judge says *"show me where…"*, find the row in §2, open that file at that
line. Nothing else is needed. Keep this open in a tab during the call.

---

## 1. The whole system in ninety seconds

A request moves through six stages. Each stage is one file. If you remember
nothing else, remember this chain:

```
student types a sentence
        │
        ▼
  app.py  /chat  (line 104)                 ← the front door
        │
        ▼
  app/ai_agent.py                            ← understands it, or refuses
        │   understand_request()  line 991
        │   answer_question()     line 790   ← for questions, not requests
        │
        ▼
  app/rag/retriever.py  search()  line 309   ← finds the verified policy
        │                                       reads knowledge_base/*.json
        ▼
  app/workflows/planner.py  build_plan()  line 498
        │   plans the steps + applies policy rules
        ▼
  ══════ HUMAN APPROVES ══════  app.py line 1275
        │
        ▼
  app/workflows/executor.py  execute_workflow()  line 172
        │   runs the plan, writes an audit row per step
        ▼
  app/tools/actions.py  TOOLS  line 334      ← the only code that
                                                changes anything real
```

**One sentence you can say out loud:** *"Understanding is in `ai_agent.py`,
the facts come from `retriever.py` reading our knowledge base, the rules and
the plan are in `planner.py`, approval is in `app.py`, and `actions.py` is the
only file allowed to change institutional records."*

---

## 2. "Show me…" → open this

The likeliest asks, in the order a judge is likely to ask them.

| If they ask… | Open | Line |
|---|---|---|
| **"Where are your snippets / knowledge base?"** | `Src/knowledge_base/` | 4 files |
| — the certificate rules | `knowledge_base/certificates.json` | `cert-001`…`cert-004` |
| — the lab rules | `knowledge_base/laboratories.json` | `lab-001`…`lab-004` |
| — the maintenance rules | `knowledge_base/maintenance.json` | `maint-001`…`maint-004` |
| — the grievance rules | `knowledge_base/grievance.json` | `griev-001`…`griev-004` |
| **"Where does it refuse to answer?"** | `app/ai_agent.py` | **709** `_unverified_answer()` |
| **"How does it find the right policy?"** | `app/rag/retriever.py` | **309** `search()` |
| **"Where's the confidence threshold?"** | `app/ai_agent.py` | **61** `CONFIDENCE_FLOOR = 0.55` |
| **"How do you stop it inventing details?"** | `app/ai_agent.py` | **455** `_is_grounded_value()` |
| **"Show me the plan it builds"** | `app/workflows/planner.py` | **433** `PLAN_TEMPLATES` |
| **"Where are the policy rules?"** | `app/workflows/planner.py` | **26–51** the constants |
| — the 24-hour lab rule | `app/workflows/planner.py` | **26** `LAB_MIN_ADVANCE_HOURS = 24` |
| — the restricted labs | `app/workflows/planner.py` | **29** `RESTRICTED_LAB_KEYWORDS` |
| — how a rule cites its source | `app/workflows/planner.py` | **187** `_note()` |
| **"Where's the approval check?"** (3 layers) | see §3 below | — |
| **"Where's the audit trail written?"** | `app/workflows/executor.py` | **22** `_audit()` |
| **"Show me the actions it can take"** | `app/tools/actions.py` | **334** `TOOLS` |
| **"Where's the double-booking check?"** | `app/tools/actions.py` | **147** `find_booking_conflict()` |
| **"Where's the role/permission system?"** | `app/security/permissions.py` | **27** `ROLE_PERMISSIONS` |
| **"How do you stop cross-department approval?"** | `app/security/permissions.py` | **166** `can_act_on()` |
| **"Where's the multilingual decline?"** | `app/ai_agent.py` | **935** `_looks_non_english()` |
| **"Which departments can it route to?"** | `app/workflows/planner.py` | **78** `DEPARTMENTS` (14) |
| **"Show me your tests"** | `Src/smoke_test.py` | 744 lines, 90 checks |

---

## 3. The approval gate — the one to know cold

This is your strongest answer, and it lives in **three separate files**. If
you memorise one thing from this document, memorise these three lines.

| Layer | File | Line | What it does |
|---|---|---|---|
| 1. The route | `Src/app.py` | **1657** `check_execution_policy()` | Refuses a non-approved request at the web layer |
| 2. The engine | `app/workflows/executor.py` | **216** | Treats `human_approval` as a gate; nothing past it runs |
| 3. **The tool itself** | `app/tools/actions.py` | **33** `_require_approved()` | Every action raises rather than acting |

**Say this:** *"It's checked in the route, again in the workflow engine, and a
third time inside each tool. Layer three is the important one — a tool must
never depend on its caller having done the check."*

Then open `app/tools/actions.py` at line 33 and show the four-line function.
It is short and it makes the point better than any explanation.

---

## 4. Every file, one line each

### `Src/` — top level

| File | Lines | What it is |
|---|---|---|
| `app.py` | 2189 | All routes, login, the request lifecycle, approval and execution endpoints |
| `smoke_test.py` | 744 | The 90 end-to-end checks |
| `seed_reviewers.py` | 136 | Creates the 10 departmental reviewer accounts |
| `create_admin.py` | 75 | Creates the admin account |
| `requirements.txt` | 7 | Flask, SQLAlchemy, Flask-Login, requests, dotenv |
| `.env` | — | Local config only. Not in git. Model, timeouts, timezone |

### `Src/app/` — the agent

| File | Lines | What it is |
|---|---|---|
| `ai_agent.py` | 1442 | Understanding, field collection, grounded answering, refusal |
| `ollama_client.py` | 83 | Talks to the local model |
| `trace.py` | 163 | Writes `logs/agent_trace.log` — *why did it decide that* |
| `formatting.py` | 135 | UTC → Asia/Kolkata for display |
| `db_migrate.py` | 277 | Adds new columns to an existing database |

### `Src/app/` — subsystems

| File | Lines | What it is |
|---|---|---|
| `rag/retriever.py` | 423 | BM25 search over the knowledge base |
| `workflows/planner.py` | 563 | Builds the plan, applies policy rules, routes to a department |
| `workflows/executor.py` | 327 | Runs an approved plan, writes audit rows, rolls back on failure |
| `tools/actions.py` | 340 | The 5 controlled actions. The only code that changes real records |
| `security/permissions.py` | 190 | Roles, permissions, departmental scope |
| `models/` | 10 files | Database tables — `request`, `approval`, `audit_log`, `user`, etc. |

### `Src/templates/` — 13 pages

`dashboard.html` is the big one (it contains the AI chat modal). Then
`approval.html`, `execution.html`, `audit.html`, `requests.html`, the four
service forms, plus `login` / `register` / `base` / `success`.

### `Src/knowledge_base/` — 4 files, 16 snippets

The verified policy. **This is the answer to "where does it get its facts".**

---

## 5. Finding anything live, without panicking

If you're asked something not on this list, run one of these in the terminal
rather than scrolling. Doing this on screen looks competent, not lost.

Find where a thing is defined:

```bash
grep -rn "def build_plan" Src/app/
```

Find every place a rule id is used:

```bash
grep -rn "lab-003" Src/
```

Find which file mentions a concept:

```bash
grep -rln "confidence" Src/app/
```

Jump to a line in your editor: `Ctrl+G` in most editors, then the number.

---

## 6. Before the call — three tabs

Have these open so you never search on camera:

1. **`Src/app/tools/actions.py`** — scrolled to line 33. The approval check.
   The single most likely "show me".
2. **`Src/knowledge_base/laboratories.json`** — the Robotics Lab rule
   (`lab-003`) is your best demo moment, and this is the file behind it.
3. **`Src/app/workflows/planner.py`** — scrolled to line 26. The policy
   constants, all visible in one screen.

Plus this file in a fourth tab.

---

## 7. If you genuinely don't know

Say so. *"I'd have to check — that part was written a few weeks ago."*

You are presenting a system whose entire argument is that it refuses to
fabricate when it isn't sure. Guessing at your own architecture in front of
professors is the one thing that would undercut the demo. Not knowing a line
number costs you nothing; inventing one costs you the room.

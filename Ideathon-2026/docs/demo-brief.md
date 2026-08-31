# Ideathon 2026 — Demo Readiness Brief

**Demo date:** 1 September 2026 (first online demo)
**Brief written:** 29 August 2026
**Repo state:** branch `tier1-grounded-agent`, commit `24c35d6`, working tree clean
**Verified for this brief:** `Src/smoke_test.py` → **90/90 checks passed**, run 29 Aug 2026

> **Standing decision — the codebase is frozen before the demo.**
> The panel is a team of inter-college professors and seniors who are expected to
> suggest what to add next. If the platform is presented as finished, there is no
> room to accept those suggestions and the session turns into a question barrage.
> Everything in the "not built" section below is **deliberate headroom**, not an
> oversight. Say so out loud during the demo — it converts a gap into a plan.

---

## The problem statement

> Develop a secure agentic-AI platform that can understand service requests, plan
> multi-step actions, retrieve verified institutional information, route approvals
> and execute permitted workflows such as certificate requests, maintenance
> tickets, laboratory bookings and grievance escalation. The system must maintain
> an auditable action trail, ask for human approval before consequential actions,
> support multilingual interaction and detect uncertainty or policy conflicts
> instead of fabricating answers.

---

## 1. Scorecard against the problem statement

The statement contains nine distinct requirements. Eight are built and working.

| # | Requirement | Status | Where it lives |
|---|---|---|---|
| 1 | Understand service requests | **Built** | `app/ai_agent.py` → `understand_request()` |
| 2 | Plan multi-step actions | **Built** | `app/workflows/planner.py` → `build_plan()` |
| 3 | Retrieve verified institutional information | **Built** | `app/rag/retriever.py` (BM25 over 16 documents) |
| 4 | Route approvals | **Built** | `planner.py` → 14 departments, `Approval.routed_to` |
| 5 | Execute permitted workflows (4 services) | **Built** | `app/tools/actions.py` → 5 controlled tools |
| 6 | Auditable action trail | **Built** | `app/models/audit_log.py`, 8 event types |
| 7 | Human approval before consequential actions | **Built** | Enforced at 3 independent layers |
| 8 | **Multilingual interaction** | **NOT BUILT** | Deliberately removed; non-English is declined |
| 9 | Detect uncertainty / policy conflicts, don't fabricate | **Built** | Confidence floor 0.55 + blocking policy notes |

**The headline: 8 of 9. Multilingual is the one real gap, and it is the single
best thing to hand the judges as "what we do next."**

---

## 2. What HAS been completed

### 2.1 Grounded answering with citations, and refusal

The agent is structurally forbidden from answering institutional questions from
model memory. Every answer must cite a snippet retrieved from
`Src/knowledge_base/*.json`. If retrieval returns nothing relevant, or the model
produces an answer citing a source that was not retrieved, the answer is thrown
away and replaced with a refusal.

- Retrieval is **BM25 over 16 verified documents** across four files
  (`certificates.json`, `maintenance.json`, `laboratories.json`, `grievance.json`),
  written using only the standard library — no vector database, no extra dependency.
- Query expansion runs before retrieval so paraphrased questions still hit
  (measured recall@4 of 5/8 vs 3/8 without it).
- The UI shows a **"📄 Verified sources: cert-002"** badge under a grounded answer,
  and a **"⚠️ Not covered by the verified knowledge base"** badge under a refusal.

**Verified live on 29 Aug:** "What is the fee for a transfer certificate?" →
*"The fee for a transfer certificate is 200 INR."*, cites `cert-002`, 4.1s.
"What is the hostel mess menu on Friday?" → refused, no sources, 4.6s.

### 2.2 Multi-step planning, shown before approval

Before anything happens, the agent builds the full ordered plan and stores it. A
reviewer sees the whole intended sequence on the approval page *before* deciding,
with the human approval gate marked in the middle and later steps shown as
not-yet-run.

Example — a laboratory booking produces six steps:

```
1  AGENT   validate_request      Check that every required detail is present
2  AGENT   check_policy          Check eligibility, notice period and restrictions
3  AGENT   check_availability    Check the slot is not already booked
4  HUMAN   human_approval        ← nothing past this line runs without a person
5  AGENT   create_booking        Reserve the laboratory slot
6  AGENT   notify_requester      Tell the requester the booking is confirmed
```

### 2.3 Policy engine that cites the rule it applied

Four evaluators (`_certificate_policy`, `_maintenance_policy`, `_laboratory_policy`,
`_grievance_policy`) read the knowledge base and attach findings to the plan. Each
finding carries the **snippet id it came from** and is either *advisory* or
*blocking*. A blocking finding marks the request as a policy conflict for the
reviewer.

Rules currently enforced:

| Rule | Source | Effect |
|---|---|---|
| Labs must be booked ≥24 hours in advance | `lab-001` | Blocking |
| Research labs need faculty co-signature | `lab-003` | Blocking, re-routes department |
| Labs cannot be double-booked | `lab-002` | Execution refused on overlap |
| Transfer Certificate needs dues cleared | `cert-002` | Advisory to reviewer |
| Character Certificate → Dean of Student Affairs | `cert-003` | Routing |
| Maintenance priority & SLA (4h / 2d / 5d) | `maint-002` | Sets priority + target |
| Harassment/safety grievances escalate in 24h | `griev-002` | High priority, Dean's office |

**Verified live on 29 Aug:** "I want to book the Robotics Lab on 5 September 2026
at 10:00 for my final year project" → filed, `policy_conflict: true`, blocking note
citing `lab-003`, routed to **Faculty Co-signature (Research Labs)** instead of
Laboratory Administration.

### 2.4 Human approval enforced at three independent layers

This is the part worth being proud of. Approval is not a UI convention — it is
checked three times by three different pieces of code:

1. **The route** — `check_execution_policy()` in `app.py` refuses a non-approved request.
2. **The executor** — will not run tool-backed steps for an unapproved request.
3. **Every tool** — `_require_approved()` at the top of each function in
   `app/tools/actions.py` raises rather than acting.

Layer 3 is the one to point at: *"a tool must never depend on its caller having
done the check."* Even if someone found a bug in the web layer, the tool itself
still refuses.

### 2.5 Real workflow execution, with rollback

Approved requests create genuine institutional records — not mock messages:

| Tool | Creates | Reference format |
|---|---|---|
| `issue_certificate` | `certificate_requests` row | `CERT-2026-0001` |
| `create_ticket` | `maintenance_tickets` row | ticket number |
| `create_booking` | `laboratory_bookings` row | `LAB-2026-0001` |
| `record_grievance` | `grievances` row | routed + prioritised |
| `notify_requester` | audit entry only | — |

If any step fails, the transaction rolls back and earlier steps are re-labelled
`ROLLED_BACK` rather than being reported as done. Nothing is left half-applied.

### 2.6 Auditable action trail

Every step writes an `audit_logs` row — including the ones that were **blocked or
refused**, which is the part that matters. Event types: `REQUEST_CREATED`,
`PLAN_CREATED`, `POLICY_CHECKED`, `APPROVAL_REQUESTED`, `TOOL_EXECUTED`,
`WORKFLOW_COMPLETED`, `WORKFLOW_FAILED`. Each row records actor type (USER / AGENT
/ Controlled Execution), the tool used, the approval status at the time, and a
timestamp. Viewable at `/audit`, also exposed as JSON at `/api/agent/audit`.

### 2.7 Uncertainty gating — refuses rather than guesses

Several independent mechanisms stop the agent inventing a request:

- **Confidence floor of 0.55.** Below it, nothing is filed and the user is asked
  to restate.
- **Fabrication penalty (−0.25).** Every field value the model invented and the
  grounding check discarded lowers confidence further.
- **Grounding check on every extracted field.** A value must actually be traceable
  to something the user typed. The model cannot invent a room number.
- **Closed-vocabulary resolution.** Certificate types and lab names are resolved
  by matching against the knowledge base, not by asking the model — so adding a
  certificate type to `certificates.json` makes it recognisable with no code change.
- **Vagueness detection.** A message built from placeholder words with nothing
  concrete in it is rejected before it reaches the model.

**Verified live on 29 Aug:** "Something happened and I am not happy about it" →
*"I'm not confident I understood that correctly, so I haven't filed anything yet."*

### 2.8 Role-based access control, scoped by department

- Five roles (`STUDENT`, `FACULTY`, `STAFF`, `REVIEWER`, `ADMIN`) in one permission
  table (`app/security/permissions.py`). Routes ask for the *permission* they need,
  never for a role name — so adding a role means editing one table.
- **Departmental scoping is real security, not navigation.** `can_act_on()` is
  enforced on the approve and execute **POST endpoints**, not just on which page a
  reviewer lands on. The Registrar cannot approve a maintenance ticket by guessing
  its request number.
- A reviewer whose department was never set can act on **nothing** — it fails
  closed, deliberately.
- 10 departmental reviewer accounts seeded via `seed_reviewers.py`; 14 departments
  defined in the router.

### 2.9 Runs fully offline on a local model

`qwen3:8b` served by Ollama on the presenter's own machine. No OpenAI, no API keys,
no student data leaving the building. All OpenAI libraries and keys were removed
earlier in the project's history. This is a genuine privacy argument for a campus
system holding grievances and student records — use it.

### 2.10 Measured performance

Timed on 29 Aug 2026 against a copy of the live database:

| Interaction | Time |
|---|---|
| Grounded question with citation | 4.1s |
| Refusal (out of knowledge base) | 4.6s |
| File a certificate request | 3.0s |
| File a maintenance request | 3.3s |
| File a lab booking with 4 fields | 4.0s |
| Non-English decline | **0.0s** (regex, no model call) |

**3–5 seconds per turn.** Comfortable for a live demo.

---

## 3. What has NOT been completed

Ordered by how likely a judge is to raise it.

### 3.1 Multilingual interaction — the one PS requirement not met

The platform is **English-only**. A message in another language is politely
declined: *"Sorry, I can only understand English at the moment."*

This was built and then **deliberately removed**. The reasoning was sound: a
half-understood service request becomes a real institutional action that someone
then has to undo. Declining was judged safer than guessing.

Detection is two-stage and cheap: a non-Latin script check (Devanagari, Tamil,
Bengali, Arabic, CJK) plus a foreign-token list covering Spanish, French, German
and romanised Hindi — then the model itself for anything subtler.

**Partly there already:** `app/rag/retriever.py` is Unicode-aware and the knowledge
base already supports aliases in Devanagari, Tamil and Bengali. The retrieval half
of multilingual exists. What is missing is understanding and replying.

**Do not hide this.** Lead with it as the next milestone.

### 3.2 CSRF protection — the biggest real security hole

There are no CSRF tokens on any POST. An authenticated admin who visits a
malicious page could be made to approve and execute a request without knowing.

This was agreed as Tier 3 work on 18 Aug and not started. It is genuinely the
correct next security task and a good answer if a judge asks "what would you
harden first?"

### 3.3 Tamper-evident audit trail

Audit rows are ordinary database rows. The trail is *complete* but not
*tamper-proof* — anyone with database access could edit history. The planned fix
is a hash chain, where each row carries a hash of the previous one, so any edit
breaks the chain visibly. Designed, not built.

### 3.4 Notifications are recorded, not sent

`notify_requester` writes an audit entry saying *"No mail service is configured,
so no message was sent."* This is honest rather than fake — but no student
actually receives anything. Wiring SMTP is a small task.

### 3.5 Credential handling

Seeded reviewer passwords are generated once and printed to the terminal. There is
no password-change UI, and no rotation. Fine for a demo, not for deployment.

### 3.6 Smaller gaps

- **No rate limiting** on the chat endpoint or login.
- **`Flask-JWT-Extended` is in `requirements.txt` but unused** — a leftover
  dependency. Sessions are used throughout.
- **`institutional_documents` table exists but is empty and has no route** —
  document upload/verification was scaffolded and never built.
- **Dead code:** `Src/static/script.js` is not loaded by any template, and the
  `/api/agent/understand` endpoint it calls is unreachable from the UI. It also
  contains a leftover `print("Received data:", data)`.
- **Knowledge base is small** — 16 documents. Enough to prove the mechanism,
  clearly not a real institution's policy corpus.
- **Legacy rows in the demo database** — see §7.1 before demoing.

---

## 4. The current goods

**The safety story is genuinely good, and it is what this problem statement is
actually about.** Most teams will build a chatbot that calls functions. This one
refuses to act when it is not sure, refuses to answer when it has no source, and
refuses to execute when a human has not approved — and each refusal is enforced in
more than one place.

1. **Approval cannot be bypassed.** Three independent layers, the innermost inside
   the tool itself. This is real defence-in-depth, not a checkbox.
2. **Every institutional claim is traceable.** Answers cite a snippet id; policy
   decisions cite the rule; the reviewer sees both before deciding.
3. **The plan is visible before approval, not after execution.** The reviewer
   approves a sequence they can read, which is the difference between oversight and
   rubber-stamping.
4. **Refusals are recorded too.** The audit trail shows what was blocked, not just
   what happened.
5. **Authorization is real.** Enforced on endpoints, scoped by department, fails
   closed on misconfiguration.
6. **90/90 automated end-to-end checks pass**, covering authentication, grounding,
   the approval gate, policy enforcement, departmental scope and authorization.
   Re-run in ~3 minutes in front of the judges if asked.
7. **Fully offline.** No third-party API, no student data leaving campus.
8. **Fast enough to demo live** — 3–5s per turn.
9. **The code is readable.** Comments explain *why*, and dead modules were deleted
   rather than left lying around.

## 5. The current bads

1. **Multilingual is missing**, and it is written into the problem statement. This
   will be noticed.
2. **No CSRF protection** — the one finding a security-minded judge could land.
3. **The audit trail is not tamper-evident**, which slightly undercuts the
   "auditable" claim if pressed hard.
4. **The knowledge base is a toy** — 16 documents. Fine as proof, obviously not
   production.
5. **Nobody is actually notified** of anything.
6. **The demo database carries legacy junk** — an empty request #8 stuck in
   "Pending Approval" with no routing, and old rows whose `user_id` are strings
   like `'67r678y'` and `'GUEST'` from before the schema settled. They render
   without crashing (verified), but they look untidy on the admin approval page.
7. **Two dead code paths** still in the tree (`static/script.js`,
   `/api/agent/understand`).
8. **Single-machine deployment.** No concurrency story, no load testing.
9. **Rules are in Python, not editable by an administrator.** Adding a *value*
   (a new certificate type) needs no code change; adding a *rule* does.

---

## 6. Questions the judges may ask, with simple answers

Keep answers to two or three sentences. Offer to show the screen rather than
explain further — the working system is the argument.

### About the AI

**Q: Which AI model is this? Are you using ChatGPT?**
No. It runs `qwen3:8b`, an open model, entirely on this laptop through Ollama.
Nothing goes to the internet, so no student data ever leaves campus. We removed
all OpenAI code and keys earlier in the project.

**Q: How do you stop it from making things up?**
It is not allowed to answer institutional questions from its own memory. It has to
find the answer in our verified policy files first and cite which file it came
from. If it can't find it, it says it doesn't know. *(Show the refusal live.)*

**Q: What if the model still hallucinates?**
Then the answer gets thrown away. We check that every source it cites was actually
retrieved — if it cites something that wasn't, we discard the answer and refuse
instead. The same applies to request details: every field it extracts has to be
traceable to words the student actually typed.

**Q: Can it invent a room number or a date?**
No. Each extracted value is checked against what the student wrote. If it can't be
traced back, it is discarded and the confidence score drops. Below 0.55 we file
nothing and ask the student to restate.

**Q: Did you train or fine-tune the model?**
No, and deliberately. Fine-tuning would bake policy into the model's weights, where
we couldn't audit or update it. Our policy lives in JSON files anyone can read and
edit — change the file and behaviour changes immediately, with no retraining.

**Q: Why is it "agentic" and not just a chatbot?**
A chatbot replies. This plans a sequence of steps, checks them against policy,
stops at a human approval gate, then executes real actions that create real records
— and logs every step. *(Show the six-step plan on the approval page.)*

**Q: How long does it take to respond?**
About three to five seconds, on this laptop, with no internet.

### About safety and approval

**Q: What stops the AI doing something harmful on its own?**
It cannot execute anything until a human approves it, and that is checked in three
separate places — the web route, the workflow engine, and inside each individual
tool. Even if someone found a bug in the website, the tool itself still refuses.

**Q: What counts as "consequential"?**
Anything that creates or changes an institutional record: issuing a certificate,
opening a ticket, booking a lab, filing a grievance. Answering a question is not
consequential, so that happens immediately.

**Q: Can a student approve their own request?**
No. Students have no approval permission at all. Requests are routed to the
department that owns them, and only a reviewer for *that* department can act.

**Q: Could a reviewer approve something outside their department?**
No. We check this on the button press, not just on the page. If the Registrar tries
to approve a plumbing ticket by typing its request number directly, it is refused.
A reviewer whose department was never configured can approve nothing at all — it
fails safe.

**Q: What is in the audit trail?**
Every step, including the ones that were refused. Who or what acted, which tool ran,
what the approval status was, and when. Blocked actions are logged too — that is
usually the part people forget.

**Q: Can the audit trail be tampered with?**
Right now it's a database table, so someone with database access could edit it. Our
next step is a hash chain — each entry carries a fingerprint of the previous one, so
any edit breaks the chain visibly. It is designed but not yet built. *(Honest
answer. Don't overclaim here.)*

**Q: What happens if a step fails halfway through?**
Everything rolls back. We don't leave a booking half-made — earlier steps get marked
as undone and the failure is logged with the reason.

### About policy and conflicts

**Q: Show me it detecting a policy conflict.**
*(This is the best demo moment — have it ready.)* Booking the Robotics Lab triggers
it: the system recognises it as a research lab, flags it as blocking, cites the rule
`lab-003`, and re-routes it to the faculty co-signature office instead of ordinary
lab administration.

**Q: Where do the rules come from?**
From our knowledge base files. Each rule the system applies names the exact policy
snippet it came from, so a reviewer can check whether the system read the rule
correctly.

**Q: Can a college change the rules without a programmer?**
Partly. Adding a new certificate type or lab is just editing a JSON file — no code
change. Adding a genuinely new *kind* of rule still needs a developer. Making the
rules fully administrator-editable is on our list.

### About the gaps — prepare for these

**Q: The problem statement asks for multilingual. Yours is English only.**
Correct, and it's our next milestone. We built it, then removed it on purpose: a
half-understood request becomes a real institutional action someone has to undo. We
chose to decline clearly rather than guess. The retrieval layer already handles
Hindi, Tamil and Bengali — the knowledge base carries aliases in those scripts. What
we still need is understanding and replying, which is why we'd rather add it
properly than half-do it.

**Q: How would you actually add multilingual?**
Detect the language, translate to English for understanding, then reply in the
original language — but only file the request if confidence stays above our
threshold after translation. The safety gate stays where it is; the language layer
sits in front of it.

**Q: What is the biggest security weakness?**
CSRF protection. An admin who's logged in and visits a malicious page could be
tricked into approving something. We've scoped the fix; it's the first thing we'd
add. *(Naming your own weakness lands better than being caught by it.)*

**Q: Only 16 policy documents? That's tiny.**
Yes — it's enough to prove the retrieval and citation mechanism works. The system
doesn't care how many there are; adding the real policy handbook is copying files
in, not writing code.

**Q: Does the student actually get notified?**
Not yet — there's no mail server wired up. We chose to log "no message was sent"
rather than pretend one was, because a fake notification in an audit trail is worse
than none.

**Q: Is this ready for real deployment?**
Not yet, and we wouldn't claim it. Before deployment we'd need CSRF protection, the
tamper-evident audit trail, real notifications, and the college's actual policy
documents. The architecture is ready; the hardening isn't finished.

### About engineering

**Q: How do you know it works?**
We have 90 automated end-to-end checks covering grounding, refusal, the approval
gate, policy enforcement, department scoping and access control. All 90 pass. It
takes about three minutes — we can run it now if you'd like.

**Q: What database?**
SQLite, which is fine at this scale. It's SQLAlchemy underneath, so moving to
PostgreSQL is a configuration change, not a rewrite.

**Q: How many people can use it at once?**
We haven't load-tested it. Right now it's one machine running one model, so the
model is the bottleneck — requests queue. Scaling means running more model
instances behind a queue.

**Q: Why not use a vector database for retrieval?**
At 16 documents it would be slower and add a dependency for no gain. We measured
keyword search with query expansion at 5 of 8 on paraphrased questions versus 3 of 8
without it. If the corpus grows to thousands of documents, we'd move to embeddings.

**Q: How long did this take, and who built what?**
*(Answer honestly with your team's split — professors ask this to check everyone
contributed.)*

**Q: What would you do with three more months?**
Multilingual, CSRF and the hash-chained audit trail first. Then real notifications,
an administrator UI for editing policy, and the college's actual policy corpus.

### Curveballs

**Q: What if a student uses it to file a fake grievance?**
Every request is tied to a signed-in account and logged. A human reviews it before
anything happens. The system doesn't judge truthfulness — it makes sure a real
person with authority does, and that there's a record.

**Q: What if the AI is confidently wrong about a policy?**
It can only quote what's in the knowledge base, and it shows which snippet. If the
snippet is wrong, the fix is editing one file. It can't be wrong in a way that isn't
traceable to a document we control.

**Q: Isn't a human approving everything just extra work?**
The human is deciding, not typing. The agent has already collected the details,
checked the rules, worked out the priority and routed it to the right office. The
reviewer reads a summary and clicks once.

**Q: Why should a college trust an AI with this at all?**
Because it isn't trusted with much. It can read policy and prepare work. Every
action that changes anything needs a person, and everything is logged. It's an
assistant to the office, not a replacement for it.

---

## 7. Demo flow

Total: **10–12 minutes** of demo, leaving room for questions.

### 7.1 Before the call — do these

- [ ] **Decide on the legacy rows.** Request #8 is an empty "Unknown" request stuck
      in Pending Approval with no routing, and requests #1–#5 have string `user_id`s
      from an older schema. They render fine but look untidy on the admin approval
      page. Either accept them, or restore from
      `Src/instance/database.db.bak-20260818-111337` — **your call, and make a
      backup first either way.**
- [ ] **Start Ollama and pre-warm the model** so the first answer isn't slow:
      ```bash
      curl -s http://localhost:11434/api/generate -d '{"model":"qwen3:8b","prompt":"hi","stream":false}' > /dev/null
      ```
- [ ] **Confirm `OLLAMA_THINK=0`** in `Src/.env` (it is). Reasoning mode roughly
      doubles every call.
- [ ] **Know your reviewer passwords.** They were generated once. If you don't have
      them, reset to a known demo value:
      ```bash
      cd Src && venv/bin/python seed_reviewers.py --password Demo!2026 --reset-passwords
      ```
- [ ] **Open two browsers** (or one normal + one incognito) — a student session and
      a reviewer session — so you never log out and back in on camera.
- [ ] **Have `/audit` open in a third tab.**
- [ ] Optionally run the smoke test beforehand so you can say "90/90 passed this
      morning" with a straight face.

### 7.2 Suggested browser layout

| Window | Signed in as | Sitting on |
|---|---|---|
| A | student (`esha` or your own account) | `/` dashboard, AI chat open |
| B | `labs` or `admin` | `/approval` |
| C | `admin` | `/audit` |

### 7.3 The run

**① Frame it — 45 seconds.** *No screen yet.*

> "Campus service requests today mean finding the right form, the right office and
> the right person. We built an agent that understands the request in plain English,
> plans what needs to happen, checks it against actual college policy — and then
> stops and waits for a human before doing anything real."

Then the one line that sets you apart:

> "The interesting part isn't what it does. It's what it refuses to do."

---

**② Grounded answer with a citation — 1 minute.** *Window A, AI chat.*

Prompt:
```
What is the fee for a transfer certificate?
```

Answers in ~4s: *"The fee for a transfer certificate is 200 INR."* — **point at the
📄 Verified sources: cert-002 badge.**

> "It didn't answer from memory. It found that in our policy file and told you which
> one. If that number is ever wrong, we know exactly which document to fix."

---

**③ The refusal — 1 minute.** *Same chat. This is your strongest early moment.*

Prompt:
```
What is the hostel mess menu on Friday?
```

Answers: *"I don't have verified information about that in the institutional
knowledge base, so I can't answer it."* — **point at the ⚠️ badge.**

> "That's not in our knowledge base, so it says so. A general chatbot would have
> invented a menu. For a campus system, being wrong is worse than being unhelpful."

---

**④ Uncertainty gate — 1 minute.** *Same chat.*

Prompt:
```
Something happened and I am not happy about it
```

Answers: *"I'm not confident I understood that correctly, so I haven't filed
anything yet."* and asks for specifics.

> "It could have filed a vague grievance and looked productive. It scored its own
> confidence below our threshold and refused. Nothing was created."

---

**⑤ File a real request — 1.5 minutes.** *Same chat.*

Prompt:
```
Water is leaking from the ceiling in Hostel B Block room 214
```

Filed in ~3s with a request number.

> "One sentence. It pulled out the location, the room and the problem — and it only
> accepted details I actually said."

Switch to **Window B** (`/approval`, signed in as admin) and show the new request:
- Classified **HIGH priority, 4-hour target**, citing `maint-002`
- Routed to **Plumbing**, with the reason written out
- The **full plan**, with the human approval gate marked

> "It has planned the whole sequence, and it's showing me the plan *before* I
> approve — not a summary afterwards."

---

**⑥ Policy conflict — 2 minutes. The best moment; don't rush it.** *Back to Window A.*

Prompt:
```
I want to book the Robotics Lab on 5 September 2026 at 10:00 for my final year project
```

Filed. Now switch to **Window B** and show:
- A **blocking policy conflict**, citing `lab-003`
- *"Robotics Lab is a research laboratory and requires faculty co-signature approval
  in addition to the standard booking."*
- Routed **not** to Laboratory Administration but to **Faculty Co-signature
  (Research Labs)**

> "Nobody told it Robotics was restricted. It read that in the policy file, flagged
> the conflict, and sent it to a different office than a normal lab booking would go
> to — and it's showing the reviewer exactly which rule it applied."

*(If you have time and want a second conflict: book any lab for a time less than 24
hours away and it will also flag the advance-notice rule, `lab-001`.)*

---

**⑦ Approve and execute — 1.5 minutes.** *Window B.*

Approve the maintenance request, go to `/execution`, and run it.

Show the step-by-step result: the ticket is created with a real number, each step
marked DONE.

> "That's a real row in the maintenance table now. Before I clicked approve, nothing
> could have created it — that's enforced in three separate places in the code,
> including inside the tool itself."

---

**⑧ The audit trail — 1 minute.** *Window C, `/audit`.*

Show the trail for the requests you just created: created → planned → policy checked
→ routed → approved → executed.

> "Every step, with who did it and when. Including the things that were refused —
> the blocked lab booking is in here too. That's the part people usually leave out."

---

**⑨ Close with the gap — 45 seconds.** *Take the multilingual question off the table
before they ask it.*

> "One thing we haven't finished. The problem statement asks for multilingual, and
> right now we're English-only — if you write in Hindi, it tells you it can only
> understand English."

*(Optionally demo it — the reply is instant.)*

> "We built it, then took it out on purpose. A half-understood request here becomes a
> real institutional action someone has to undo, so we chose to decline clearly
> rather than guess. The retrieval layer already handles Hindi, Tamil and Bengali —
> what's missing is understanding and replying, and that's our next milestone. The
> other two we'd add before anyone deployed this are CSRF protection and a
> tamper-evident audit log."

> "We'd genuinely like your input on what to prioritise after that."

*(That last line is the whole point of the frozen codebase. It invites advice
instead of interrogation.)*

### 7.4 If something goes wrong

| Problem | Do this |
|---|---|
| Model slow or not responding | Check Ollama is up. Fall back to the manual forms at `/certificate`, `/maintenance`, `/laboratory`, `/grievance` — they run the **same** planner, policy engine, approval routing and audit trail, just without the chat. Nothing in the story is lost except the language understanding. |
| An answer comes back oddly worded | Don't fight it. "The model phrasing varies — what matters is the source badge underneath, which is fixed." Move on. |
| A request doesn't file in one turn | That's the design. Answer its follow-up question — it collecting missing details *is* a feature worth narrating. |
| A page errors | Go to `/audit` and keep talking about the trail. All nine pages were verified rendering on 29 Aug, so this is unlikely. |
| Asked something you don't know | "I don't know — I'd have to check." Given a demo built entirely around refusing to fabricate, this answer is thematically perfect. Do not guess. |

### 7.5 Things NOT to say

- Don't say "fully secure" or "production ready" — CSRF and the audit chain aren't done.
- Don't say "it can't hallucinate" — say it can't hallucinate *institutional policy
  without being caught*, because every claim must cite a retrieved source.
- Don't oversell the knowledge base. 16 documents. Say so.
- Don't claim multilingual works in any form. Retrieval is Unicode-aware;
  conversation is English-only.

---

## 8. One-line summary if you only have thirty seconds

> A campus service agent that understands a request in plain English, plans the
> steps, checks them against real college policy and cites the rule, routes it to
> the right office, and refuses to do anything consequential until a human approves —
> logging every step, including the ones it refused. Runs entirely offline on the
> college's own machine.

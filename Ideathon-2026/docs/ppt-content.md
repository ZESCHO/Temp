# PPT content — SIH 2026 idea submission template

Drafted against `SIH2026-IDEA-Presentation-Format.pptx`.

**Template rules, from the instructions slide:**

- **Six slides maximum, including the title slide**
- Use the provided template; **do not change the section headings**
- No paragraphs — points, diagrams, infographics, pictures
- Delete slide 7 (Important Instructions) before uploading
- **Export to PDF.** The portal accepts nothing else

Everything below is written to be pasted as bullets. Keep it terse — if a
line wraps twice on the slide, cut it.

---

## Slide 1 — Title page

Fill from the portal; these four are yours to look up:

- **Problem Statement ID** — `<from portal>`
- **Problem Statement Title** — Secure agentic-AI platform for campus service
  requests, approvals and workflow execution
- **Theme** — `<from portal — likely Smart Automation>`
- **PS Category** — Software
- **Team ID** — `<from portal>`
- **Team Name** — `<registered name — must not contain the college name>`

---

## Slide 2 — Proposed solution

**Idea title:** An AI agent that runs campus services end to end — and refuses
to act without a human.

**The solution**

- Students describe a need in plain English; the agent handles the rest
- Four services: certificates, maintenance tickets, lab bookings, grievances
- Answers only from the college's verified rulebook, citing the exact rule
- Plans every step, then **stops for human approval** before acting
- Executes the approved workflow and records a real institutional action

**How it addresses the problem**

- Replaces form-hunting and office-hopping with one entry point
- Reviewers receive requests already checked, prioritised and routed
- Every answer and decision is traceable to a policy document

**Innovation and uniqueness**

- **Refuses instead of fabricating** — no verified source, no answer
- Cites the exact rule it applied, so a reviewer can audit the reasoning
- Approval enforced at **three independent layers**, the innermost inside the
  action itself
- **Runs fully offline** on a local model — no student data leaves campus

---

## Slide 3 — Technical approach

**Technologies**

- Python 3, Flask, SQLAlchemy, SQLite
- **Qwen 3 (8B) served locally via Ollama** — no cloud API, no keys
- BM25 lexical retrieval with LLM query expansion (standard library only)
- Rule-based policy engine; role-based access control with departmental scope

**Methodology — the request lifecycle** *(draw this as the slide's diagram)*

```
Request → Understand → Retrieve verified policy → Plan steps
        → Check policy → ■ HUMAN APPROVAL ■ → Execute → Audit
```

- **Understand** — extract fields; every value must trace to the user's words
- **Retrieve** — 16 verified policy snippets; answer or refuse
- **Plan** — ordered steps shown to the reviewer *before* approval
- **Policy** — flags conflicts and routes to the owning department
- **Approve** — nothing consequential runs until a person approves
- **Execute** — controlled tools write the real record; rollback on failure
- **Audit** — every step logged, including refusals

**Status: working prototype, 90/90 automated end-to-end tests passing.**

---

## Slide 4 — Feasibility and viability

**Feasibility — already demonstrated**

- Working prototype, not a concept: 90 automated end-to-end checks pass
- Runs on one laptop with no internet — zero infrastructure cost
- 3–5 second response time on commodity hardware
- Adding the college's real policy handbook is copying files, not coding

**Challenges and risks**

- Multilingual interaction not yet supported (English only)
- Web-layer hardening incomplete (CSRF protection)
- Audit trail is complete but not yet tamper-evident
- Keyword retrieval will not scale to a very large policy corpus

**Strategies to overcome**

- Language layer sits **in front of** the safety gate, so thresholds still apply
- CSRF tokens on all state-changing requests
- Hash-chained audit rows — any edit breaks the chain visibly
- Move to embedding-based retrieval as the corpus grows

---

## Slide 5 — Impact and benefits

**Target audience: students, reviewing staff, the institution.**

**Students**

- One place to ask, in plain language; no form-hunting between offices
- Verified answers on fees, deadlines and eligibility in seconds
- Grievances routed confidentially and escalated on time

**Staff and reviewers**

- Requests arrive pre-checked, prioritised and routed to the right office
- The applicable rule is cited, so decisions take a click, not an investigation

**Institutional**

- Complete auditable trail of every action, including refusals
- Accountability: separation between what the AI prepares and what a human decides

**Social** — harassment and safety grievances bypass the chain being complained
about and escalate to the Dean within 24 hours

**Economic** — no per-query API cost; runs on existing hardware

**Privacy** — student records and grievances never leave campus

---

## Slide 6 — Research and references

- Institutional policy corpus — the college's own certificate, maintenance,
  laboratory and grievance rules (the system's only source of truth)
- **Retrieval-Augmented Generation** — Lewis et al., 2020, *Retrieval-Augmented
  Generation for Knowledge-Intensive NLP Tasks*
- **BM25 ranking** — Robertson & Zaragoza, 2009, *The Probabilistic Relevance
  Framework: BM25 and Beyond*
- **Qwen 3** — open-weight model family, Alibaba
- **Ollama** — local model serving, ollama.com
- Project repository and technical documentation — `<add your GitHub link>`

> Verify every link before submitting. Add any college policy documents you
> actually used.

---

## Before you upload

- [ ] Six slides or fewer, title included
- [ ] Instructions slide deleted
- [ ] Section headings unchanged from the template
- [ ] No paragraphs — bullets, diagrams and screenshots only
- [ ] Slide 3 carries the lifecycle diagram; consider a screenshot of the
      approval screen showing a cited policy rule
- [ ] Team name does not contain the college name
- [ ] **Exported as PDF**

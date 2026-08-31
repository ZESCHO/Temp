# Answers to drill

From the mock panel on 29 Aug. These are the six where the instinct was right
but the answer needed sharpening. Read them out loud once; that is enough.

---

## 1. The harassment grievance — get this one exact

**Verified against the code**, not remembered:

```
"my hostel warden is harassing me"
  → priority   HIGH
  → routed to  Dean of Student Affairs      (NOT the warden, NOT hostel admin)
  → escalated  within 24 hours
  → cites      griev-002
  → asks for   subject + description only   (NOT hostel number or warden name)
```

> "It classifies it as a grievance, and the word 'harassing' triggers our
> high-priority rule — that's `griev-002`. It routes straight to the Dean of
> Student Affairs and flags it for escalation within 24 hours, deliberately
> bypassing the hostel chain, because a complaint about a warden shouldn't go
> to the warden's office. Only Dean of Student Affairs reviewers can see it —
> departmental scoping means no other office in the system can open it."

The bypass is the point. Don't lose it.

---

## 2. Multilingual

Not "we ran out of time." It was a decision.

> "We built it, then took it out on purpose. A half-understood request here
> doesn't produce a bad chat reply — it produces a real institutional action
> someone has to undo. We decided declining clearly was safer than guessing.
> The retrieval layer already handles Hindi, Tamil and Bengali, so it's
> half-built; what's missing is understanding and replying, and that's our
> next milestone."

---

## 3. "What stops a student approving their own request?"

Name the mechanism, not the future plan.

> "A student can't approve anything — the permission doesn't exist for that
> role. Reviewers only see and act on their own department's queue, and that's
> checked on the endpoint, not just the page. The admin account you're
> watching is a master role we're using to demo both sides quickly."

**Known gap, volunteer it if pressed:** there is no self-approval guard. A
reviewer could approve their own request if it routes to their own department.
Separation of duties is a real gap and a small fix.

---

## 4. "Anything insecure?"

Lead with CSRF, not registration.

> "Two we know about. There's no CSRF protection, so a logged-in admin
> visiting a malicious page could be tricked into approving something — that's
> the first thing we'd fix. And registration blocks duplicate registration
> numbers but doesn't verify the number belongs to you; that needs checking
> against the student register."

---

## 5. "How much did the AI write?"

Don't undersell. You made the calls.

> "AI accelerated the implementation. The design decisions are ours and I can
> defend each one — why we refuse below a confidence threshold instead of
> filing and flagging, why we removed multilingual rather than ship it
> half-working, why there's one login instead of a separate admin path. Happy
> to go into any of them."

Then stop talking.

---

## 6. "Only 16 documents — isn't this a toy?"

Never concede the frame. Small corpus, not a small system.

> "The corpus is small, the mechanism isn't. Nothing in the system depends on
> there being sixteen — adding your handbook is copying files into a folder,
> not writing code. What we've proved is that it retrieves the right rule,
> cites it, and refuses when the answer isn't there. At a few thousand
> documents we'd switch keyword search for embeddings — that's a retrieval
> change, not a redesign."

Optional: *"We'd genuinely like a copy of the handbook to load in."*

---

## Two other corrections

- **Qwen is not a Llama model.** Qwen is Alibaba's; Llama is Meta's. Ollama is
  the runner. Say: *"Qwen 3, an open-source model, running locally through
  Ollama."*
- **The snippets are in `Src/knowledge_base/`**, not in the `rag` folder.
  `app/rag/retriever.py` is the code that searches them. The rag folder is the
  librarian; `knowledge_base` is the shelf.

---

## The one habit

Every miss last night was the same shape: reaching for a **process** answer
when a **mechanism** answer was available and stronger.

When asked "what stops X" — name the check, not the intention.

---
---

# Round 2 and 3 — added 30 Aug

## The one line that fixes the recurring miss

Three times the wrong guard was reached for. There are two, and this covers both:

> **Questions get answered from sources. Requests get filled from your own words.**

- Asked how it won't invent a **fee** → it must find it in the rule files and cite it; if it isn't there it refuses.
- Asked how it won't invent a **room number** → the value must appear in what the student typed, or it's discarded and confidence drops.

Do not answer either of these by opening `logs/agent_trace.log`. That is a
debugging file, unreadable on a projector. The audit trail in the UI is what
you show.

---

## DEMO HAZARD — follow-up phrasing (verified 30 Aug)

Answering a follow-up question with a bare value **switches the request type**.

| Follow-up | Result |
|---|---|
| `Lab 3` | switches to a lab booking, maintenance request lost |
| `the room number is 12` | forgets the location, asks again |
| `room 12` | files correctly |
| `room 12, chemistry lab` | files correctly |

**Always repeat the noun: "room 12", never "12" or "Lab 3".**
Safest of all: drive the keyboard yourself.

---

## Where the rules live — two places

**The rulebook — `Src/knowledge_base/`** (4 files, 16 rules). Fees, deadlines,
eligibility, who issues what. Editing these needs no programmer.

**The enforcement — `Src/app/workflows/planner.py`, lines 26–78.** The 24-hour
lab notice, restricted labs, priority keywords, department routing. This is code.

> "Changing a fee is editing a file. Changing something like the 24-hour
> booking notice is a code change."

Two traps: there is **no admin screen** for editing the rulebook — it is a text
file on the server. And the knowledge base is **cached at startup**, so a live
edit needs a restart ("about two seconds").

It is `Src/knowledge_base/`. Not the `rag` folder. Not `app/knowledge_base`.

---

## "Why not just upload the rulebook to ChatGPT?"

The most likely question in the room.

> "ChatGPT can tell you the fee. It can't issue the certificate. Ours doesn't
> just answer — it opens the ticket, routes it to Plumbing, books the lab, and
> waits for a human to approve before any of that becomes real.
>
> And we couldn't upload our rulebook anyway. This handles grievances and
> student records — that data can't leave campus, which is why the model runs
> on this laptop with no internet.
>
> One more thing: ask ChatGPT something your rulebook doesn't cover and it will
> usually still answer. Ours says it doesn't know."

Short version: **"ChatGPT can tell you the fee. It can't issue the certificate."**

---

## "What stops another team building this in a weekend?"

> "A weekend gets you a chatbot that calls functions — that part is genuinely
> easy now. What took the time was everything that stops it doing damage:
> citing the rule it used, refusing when the answer isn't in our files,
> discarding details the student never said, and making sure nothing reaches a
> real record without a human approving it."

Short version: **"A weekend gets you a chatbot. What took us time was making it refuse."**

---

## "You wrote your own tests — doesn't that prove nothing?"

Concede, then reframe, then invite them to break it.

> "That's fair — tests we wrote can't prove the idea is good. What they prove is
> that the safety behaviour holds every time, not just in a demo. Most of them
> test things it must *refuse* to do.
>
> But the proof you should want is better than our tests. Ask me to make it do
> something it shouldn't — approve without a human, or answer something that
> isn't in our rules."

---

## "Hardest thing you personally solved?"

Name the trade-off, not just the feature.

> "The refusal logic. Refusing is easy — knowing *when* is the hard part. Too
> strict and it rejects real requests; too loose and it files nonsense. We
> ended up with a confidence threshold plus a check that every extracted detail
> appears in what the student typed. Tuning that took the longest."

---

## "Does it learn from students as they use it?"

Wrong premise. Correct it, then follow through:

> "No — it's pre-trained and it doesn't learn from users. It answers from our
> rule files, not from its own memory. It improves when we edit those files,
> which takes effect immediately and we can see exactly what changed. If it
> learned from students, one wrong answer could spread to everyone and nobody
> could tell you why."

---

## "What is Qwen 3, 8B?"

> "A model we downloaded, running locally through Ollama — no external API. 8B
> is 8 billion parameters, basically its size: big enough to be capable, small
> enough to run on this laptop."

---

## The 60-second walkthrough, login to approval

Beats: **sign in → type it → it extracts → it checks the rules → it plans, and stops**

> "A student signs in with their username or registration number, so nothing is
> anonymous. They type what they need in plain English — 'water is leaking in
> Hostel B, room 214.' The agent works out it's a maintenance request and pulls
> out the block, the room, the problem — accepting only details the student
> actually typed. If something's missing it asks instead of guessing.
>
> Then it checks the request against our rulebook: high priority, four-hour
> target, belongs to Plumbing — and it records which rule said so. It writes out
> the full plan of what would happen, and stops.
>
> The request lands in the Plumbing reviewer's queue with the plan and the rule
> attached, waiting for a person to approve. **Nothing has been created yet.**"

Land that last line. Cut anything else before you cut it.

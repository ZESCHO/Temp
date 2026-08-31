# Eval 1 demo script — 7 minutes, 6 speakers

**Sept 1, 19:00–21:30. 15 minutes total.** Demo ~7 min, leaving 8 for the
judges to ask and suggest. Mentorship round — they cannot eliminate you here.

**Audience:** professors and seniors, not working engineers. Most of the other
289 teams have no prototype. Your advantage is not depth, it is that the thing
runs.

---

## The rule that governs everything below

> **Say something that sounds technical → immediately show that exact thing
> on screen.**

Close that loop every time and each claim buys credit for the next one. Break
it once — claim something they cannot see — and everything afterwards gets
quietly discounted.

So: never say a sentence you cannot point at.

---

## The one sentence they must leave with

Say it in the opening. Say it again at the policy conflict. Say it in the
close.

> **"It reads the college's own rules, and it cannot do anything without a
> human approving it."**

If a judge can repeat that to another judge afterwards, you are in the top 100.
Everything else is supporting evidence.

---

## One technical term each

Each speaker owns **one** term. Say it once, restate it in plain words
immediately, then show it. Do not stack jargon — one term per person is what
sounds confident; three per person sounds like hiding.

| # | Speaker | Their term | The plain restatement |
|---|---|---|---|
| 1 | Opener | "agentic AI" | "it plans and acts, it doesn't just chat" |
| 2 | Q&A | "grounded in a knowledge base" | "it can only answer from our rule files" |
| 3 | Refusal | "hallucination" | "AI confidently making things up" |
| 4 | Conflict | "policy engine" | "it checks the request against the rules" |
| 5 | Approval | "human-in-the-loop" | "a person has to approve before anything happens" |
| 6 | Audit | "audit trail" | "a record of every step, including refusals" |

"Hallucination" is the strongest word in the list — they have heard it in the
news, so it lands, and using it correctly signals you know the field.

---

## The run

### ① Opening — 45 s — *no screen yet*

> "A student who needs a bonafide certificate today has to find the right form,
> the right office and the right person, and usually gets sent back at least
> once. We built an AI agent that handles the whole request — but the important
> word is *agentic*: it doesn't just chat, it plans the steps and carries them
> out. **It reads the college's own rules, and it cannot do anything without a
> human approving it.**"

Then hand over. Do not explain the architecture. They will not retain it.

---

### ② A question it can answer — 60 s

Type:

```
What is the fee for a transfer certificate?
```

Answer comes back in ~4 seconds: *"The fee for a transfer certificate is
200 INR."*

**Point at the source badge on screen.**

> "See that tag underneath — `cert-002`. That's the exact rule it read. It's
> **grounded in our knowledge base**, which just means it can only answer from
> the college's own rule files, and it shows you which one. If that fee ever
> changes, we edit one file — we don't retrain anything."

*(Loop closed: said "grounded", showed the badge.)*

---

### ③ The refusal — 60 s — **your strongest 60 seconds**

Type:

```
What is the hostel mess menu on Friday?
```

Answer: *"I don't have verified information about that in the institutional
knowledge base, so I can't answer it."*

> "That's not in our rules, so it refuses. You've probably read about AI
> **hallucination** — confidently making things up. A normal chatbot would have
> invented a menu and sounded certain. For a campus system, a confident wrong
> answer about fees or deadlines is worse than no answer."

Pause here. Let it sit. This is the moment that separates you from a chatbot
in their minds, and non-experts understand it immediately.

---

### ④ The policy conflict — 2 min — **the centrepiece, do not rush**

Type:

```
I want to book the Robotics Lab on 5 September 2026 at 10:00 for my final year project
```

It files. Switch to the reviewer screen.

Point at three things **in this order**:

1. **The blocking flag** — "it stopped this one."
2. **The reason and its source `lab-003`** — *"Robotics Lab is a research
   laboratory and requires faculty co-signature."*
3. **The department** — routed to **Faculty Co-signature (Research Labs)**, not
   ordinary Laboratory Administration.

> "Nobody programmed 'Robotics' into the code. It read that rule in our policy
> file, worked out this booking needs a faculty signature, and sent it to a
> different office than a normal lab booking would go to. That's the **policy
> engine** — it checks every request against the rules before a human ever sees
> it, and it tells the reviewer exactly which rule it applied."

Then repeat the sentence:

> "It reads the college's own rules."

---

### ⑤ Approval and a real record — 75 s

Approve it. Execute it. Show the created record with its reference number.

> "Nothing happened until I clicked approve. That's **human-in-the-loop** — the
> AI prepares the work, a person makes the decision. And that's not just the
> button being hidden; the check is enforced in three separate places in the
> code, including inside the function that creates the record. Even if someone
> got past the website, the action itself refuses."

If — and only if — a judge looks interested, open `app/tools/actions.py` at
line 33 and show the four-line check. It is short enough to read on screen.
Otherwise keep moving.

---

### ⑥ Audit trail and close — 60 s

Open the audit page.

> "Every step is recorded — who did it, when, and what was decided. Including
> the things it refused. That blocked lab booking is in here too."

Then close, and **invite the suggestions** — this is a mentorship round and
asking for advice is what it is for:

> "One thing we haven't finished: the problem statement asks for multilingual,
> and right now it's English-only. We built it and took it out on purpose — a
> misunderstood request here becomes a real action someone has to undo, so we
> chose to decline clearly rather than guess. It's our next step.
>
> We'd genuinely like your input on what to prioritise after that."

---

## What NOT to say to this panel

Save these unless asked directly. They add nothing here and risk losing the
room:

- BM25, embeddings, vector databases, recall@4
- Confidence thresholds and penalty scores
- CSRF, hash chains, schema details
- Anything about file structure or line numbers

If they ask a genuinely technical question, answer in one plain sentence and
offer to show it. Depth on request, never volunteered.

---

## If you don't understand a question

> "Could you say a bit more about what you'd like to see?"

Nine times out of ten they will restate it more simply and answer it for you.
Do not guess at a question you didn't parse — guessing is the one thing that
breaks the trust loop you spent seven minutes building.

---

## Two practical things

**Assign the recorder now.** One member's only job during the Q&A is writing
down the judges' suggestions in their own words. Eval 2 is at 07:30 the next
morning and rewards showing those suggestions integrated. Nobody will remember
the wording at 11 PM.

**All 6 must take part.** The order above gives everyone a beat and a term.
Rehearse the handoffs — "over to you, X" — because six people improvising
transitions on a video call is where the polish gets lost, not in the content.

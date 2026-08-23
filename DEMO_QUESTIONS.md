# Demo question set

Every question below was run against the live backend and verified. Response sizes are the
measured character counts, so nothing here produces a one-line answer.

**Type them in order, in a single chat session** — turns 11 and 12 only work because the
earlier turns are still in context.

Set `user_id=ayush` when the session is created (the frontend already does) so the order-tracking
answer picks up the mock profile and cites order **NC-10234**.

---

## The main flow (14 turns, ~6 minutes)

| # | Type this | What to point out | Reply size |
|---|---|---|---|
| 1 | `hey there, i'm Ayush — what can you help me with?` | Greets **by name**, introduces itself, lists what it covers | 198 ch |
| 2 | `how do i track my order?` | Pulls order **NC-10234** from the mock customer profile | 287 ch |
| 3 | `and how long does delivery usually take?` | "that" / "and" resolved from turn 2 — no repeated question | 498 ch |
| 4 | `what is your return policy?` | Grounded: **7 calendar days**, unused condition | 312 ch |
| 5 | `what items cannot be returned?` | Longest policy answer — exclusions + the damaged-item carve-out | **604 ch** |
| 6 | `when will i get my refund after i return something?` | Refuses to invent a number of days, points to Refund Status | 484 ch |
| 7 | `how do i cancel my order?` | Numbered steps + what happens past the cutoff | **596 ch** |
| 8 | `what if my item arrives damaged?` | **48-hour** window, photos, manual review | 339 ch |
| 9 | `what payment methods do you accept?` | UPI / cards / net banking / COD | 307 ch |
| 10 | `what is NovaCart Plus membership?` | Strongest retrieval match in the KB (0.875) — **₹99/mo, ₹899/yr** | 423 ch |
| 11 | `what's my name and what did i ask you first?` | **Memory** — recalls "Ayush" and turn 2, ten turns later | 163 ch |
| 12 | `my order never arrived and nobody has responded, this is unacceptable` | **Sentiment -0.83** → answers first, *then* offers a human | 212 ch |
| 13 | `what is the capital of france?` | **Out of scope** — declines to guess instead of hallucinating | 297 ch |
| 14 | `okay please connect me to a human agent` | **Escalation** — handoff + "Handed to agent" badge | 162 ch |

---

## Optional extras

Swap in if you need more material, all verified strong:

- `how do i change my delivery address?`
- `my package is delayed, what should i do?`
- `how do i report a damaged product?`
- `my order says delivered but i have not received it`
- `how do i return a product?`

## Escalation edge cases (worth 30 seconds)

Shows the handoff is precise, not keyword-triggered:

| Type this | What happens |
|---|---|
| `is there a human review step for damaged item claims?` | **Answers it.** Contains "human" but isn't a request for one |
| `the courier agent left a note at my door, what do i do?` | **Answers it.** Contains "agent", still not a handoff |
| `put me through to a real person please` | **Escalates.** Different words, same intent |

## Backend proof (after the chat)

The session id is shown at the top of the chat and is copyable.

```bash
curl "http://localhost:8000/logs?session_id=<PASTE_SESSION_ID>"
```

Every turn is logged with role, intent, sentiment, escalation flag, and timestamp. Thumbs
up/down on any message writes to the feedback table; `python scripts/export_review_queue.py`
exports the thumbs-down ones.

---

## Do not use these on camera

Verified to fall flat:

| Avoid | Why |
|---|---|
| `the app keeps crashing when i open my cart` | Retrieval scores **0.268**, under the 0.30 threshold — falls back to "I don't have that" |
| `do you ship internationally and what does it cost?` | Not in the KB; the honest refusal is correct behaviour but looks like a failure on video |
| `honestly this is unacceptable, i've been waiting weeks...` | "honestly" reads as **positive** to VADER, dragging the score to -0.44 — misses the -0.5 threshold and the human offer never appears. Use the turn-12 wording instead |
| Anything about loyalty points, gift cards, or warranty | Not in the knowledge base |

## Before you record

```bash
cd backend
GROQ_MODEL="openai/gpt-oss-120b" .venv/Scripts/python.exe -m uvicorn app.main:app --port 8000
```

Wait for `Application startup complete` (~15 s, it loads the embedding model), then start the
frontend with `npm run dev`.

Groq's free tier is capped at **200,000 tokens/day** per model. A full rehearsal plus the real
take is roughly 30 turns, which is fine — but if replies suddenly go empty or return "having
trouble reaching the support service", that's the daily cap, not a bug. Switch `GROQ_MODEL` in
`.env` to another model (each has its own quota) and carry on.

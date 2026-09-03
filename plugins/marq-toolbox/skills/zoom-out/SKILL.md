---
name: zoom-out
description: Comprehension reset for when a coding agent went too deep or too technical. Use whenever the user signals they lost the thread of an explanation — "/zoom-out", "zoom out", "back up", "too deep", "too in the weeds", "I don't understand", "what does that even mean", "you lost me", "ELI5", "explain it simply", "in plain English", "why are we even here", "what is X", "lost the plot", "that's a lot of jargon" — or any sign the previous response assumed knowledge of a system, term, or mechanism the user doesn't have. Not for "too long" complaints (that's the tldr skill); this is for "too deep / too technical". Rebuilds the explanation as a guided descent from the user's original goal down to the specific point, defining every term as it appears, then pauses. Trigger even on casual or frustrated phrasing ("huh?", "wait what", "ok I'm not a real engineer, help").
---

# zoom-out — climb back to the goal, then descend slowly

The user asked for something, and somewhere along the way you ended up several layers deep in a
mechanism they don't have a mental model for, speaking in terms they don't own. They aren't asking
for less information. They're asking for the *staircase* they never got: the sequence of ideas that
connects the thing they asked for to the thing you're now talking about.

Your job is to rebuild that staircase, one step at a time, in language a smart person from
outside the field would follow. Then stop.

This is not a request to redo work, run more tools, or continue the task. Change the altitude and
the vocabulary of the explanation, not the substance.

## Who you're explaining to

Assume a technically confident non-engineer: comfortable with business, SaaS, data, and systems
thinking; able to read code but not fluent in the idioms of any one stack; unfamiliar with the
specific subsystem you dove into. Simplify the *language*, never the *facts*. If a simplification
is lossy, say so in one clause ("roughly — the real mechanism has an extra step") rather than
either lying or dumping the whole truth.

Plain language is not baby talk. Don't be cute, don't pad, don't apologize.

## How to build the descent

1. **Find the anchor.** Identify the last thing the user clearly understood — usually their
   original request or goal, stated in their own words. Start there.
2. **Find the destination.** Identify the specific deep point you were explaining when they got
   lost. If the user passed an argument (`/zoom-out the OAuth part`), that's the destination.
3. **Lay out 3–5 levels between them.** Each level narrows the scope: whole system → the part
   that matters → the mechanism inside that part → the specific thing that's going on. If you
   need more than five, you are probably explaining two different descents; pick the one that
   matters for the decision at hand and mention the other exists.
4. **At each level, do three things:**
   - Say what this thing *is* and what it's *for*, in one or two sentences.
   - Define every term of art the moment it appears, in the same sentence, in plain words. If
     you'd have to define three new terms to explain one, you skipped a level.
   - Give a concrete analogy or example when the concept is unfamiliar. Keep it short and keep it
     next to the real explanation, not instead of it.
5. **Explain the turn.** At each step, say *why* the path goes here and not somewhere else: what
   about the previous level led you to look at this one. That's the narrative thread the user
   lost, and it's what makes the deep point feel inevitable rather than arbitrary.
6. **Land the plane.** At the bottom, restate the deep point in the new vocabulary, and connect
   it back to the anchor: what this means for what the user actually asked for, and what decision
   (if any) hangs on it.

## Output format

```
**Where we're headed:** <one sentence: the deep point, stated as plainly as possible>

**Level 1 — <the whole thing, in the user's terms>**
<2–4 sentences. Define any term as it appears.>

**Level 2 — <the part that matters>**
<2–4 sentences. Why we went here from Level 1.>

**Level 3 — <the mechanism>**
<2–4 sentences. Analogy if useful.>

**Level 4 — <the specific point>**
<2–4 sentences. The deep point in plain language.>

**So what:** <what this means for the user's original goal, and any decision waiting on them>

**Terms used:** <term> — <plain definition>; <term> — <plain definition>; ...

Reply with a level number to go deeper there, a term to expand, or "go" to continue.
```

Rules that keep this useful:

- **Each level ≤ ~80 words.** The whole descent should fit on one screen. Depth is available on
  request; it isn't the default.
- **One new idea per level.** If a level needs two unfamiliar concepts, split it.
- **No unexplained jargon anywhere**, including in the level titles. If a term is unavoidable
  (a product name, a real command), define it inline and list it under "Terms used".
- **No code, paths, or commands in the levels** unless the user's confusion is specifically about
  a piece of code. Even then, quote the smallest fragment that makes the point.
- **"So what" is mandatory.** The user asked why they should care; the descent isn't done until
  it says what changes for them.

## After the descent

- If the user replies with a level number, expand only that level: more detail, a second
  example, or the lossy-simplification caveat spelled out. Keep it to roughly a screen.
- If they reply with a term, define it more fully and show where it appeared in the original
  explanation.
- If they ask "why not X", answer at the same altitude they're asking from; don't dive again.
- If they say "go", resume the task where you left off.
- Don't proceed with the task until they say so. The whole point is that they want to understand
  before anything else happens.

## Example

Prior turn: while fixing a report that double-counted revenue, you explained that "the incremental
model's merge strategy keys on a surrogate hash of (account_id, period), so late-arriving facts
from the CDC stream land as new grain rows instead of updating in place, and the downstream
snapshot picks up both."

Descent:

```
**Where we're headed:** Some revenue rows are getting added twice because the table that stores them can't tell an "update" from a "new row".

**Level 1 — The revenue report**
The report reads from one big table of revenue-by-account-by-month. If that table has a duplicate row, the report double counts. That's the symptom you noticed.

**Level 2 — How the table gets refreshed**
Rebuilding the whole table nightly would be slow, so it's built "incrementally": each night only the new or changed rows are added. To do that, the build needs a way to recognize "I've seen this row before" — that's called a key.

**Level 3 — What the key is made of**
Here the key is a fingerprint of two fields: which account, and which month. Two rows with the same account and month should be the same row, and a refresh should overwrite the old one rather than add a second.

**Level 4 — Why the fingerprint fails**
Some source updates arrive days late, after the month's row already exists. The build treats a late update as a brand-new row instead of a replacement, because of how the "overwrite vs. add" rule is configured. Result: two rows, same account, same month.

**So what:** The fix is a one-line config change telling the build to overwrite on matching fingerprint. Before I do that, you should know it will rewrite the affected rows on the next run, which will change last month's reported total by a small amount.

**Terms used:** incremental model — a table rebuilt by adding only what changed; key — the field(s) used to recognize a row as already seen; late-arriving facts — updates that show up after the row they belong to was already written.

Reply with a level number to go deeper there, a term to expand, or "go" to continue.
```

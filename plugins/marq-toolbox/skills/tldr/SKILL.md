---
name: tldr
description: Cognitive-load circuit breaker. Use whenever the user signals they just received too much information at once — "/tldr", "tl;dr", "too much", "TMI", "wall of text", "I'm lost", "slow down", "overwhelmed", "bullet points please", "summarize what you just said", "consolidate that", "what do I actually need to know", "where are we" — or any complaint that a previous response (or a run of tool calls plus a final response) was too long to digest. Rewrites the recent output as a short numbered digest the user can drill into one item at a time, and pauses all further work until they choose. Trigger even when the wording is casual or frustrated ("ok that was a lot", "my brain hurts", "wtf did you just do").
---

# tldr — consolidate, then wait

The user just got buried. Whatever you (or a prior turn) produced was more than they can hold in
their head at once. Your only job now is to shrink it to something scannable, make every detail
reachable on request, and stop.

Do not treat this as a request to redo the work, re-run tools, or "proceed with the next step."
It is a request to change the *shape* of the information, not to generate more of it.

## What to summarize

Cover everything since the user's last substantive message — not just your final answer. In
agentic sessions the important facts often live between tool calls: a value discovered mid-run,
an error that was silently worked around, a decision you made on the user's behalf, a file you
changed. Those pockets are exactly what the user could not track while scrolling past tool
output, so sweep the whole stretch: your interstitial text, notable tool results, and the final
response.

If the user passed an argument (e.g. `/tldr what broke` or `/tldr the pricing part`), treat it as
a focus: still produce the digest, but weight it toward that topic.

## Output format

Reply with this and nothing else:

```
**Bottom line:** <one sentence: where things stand right now>

1. [tag] <point, ≤ 15 words>
2. [tag] <point>
3. [tag] <point>
...

**Needs you:** <the single decision or answer that blocks progress, or "nothing — waiting on your go-ahead">

Reply with a number to expand, "all" for full detail, or "go" to continue.
```

Rules that make this work:

- **3–7 points.** If you have more, merge or drop the least consequential. Seven is the ceiling
  because the goal is a list the user can hold in working memory, not a table of contents.
- **Tags** tell the user what kind of item it is without reading it: `[done]`, `[found]`,
  `[changed]`, `[decision]`, `[blocked]`, `[risk]`, `[question]`. Put `[decision]`,
  `[question]`, and `[blocked]` items first — those are the ones that gate progress.
- **Plain language only.** No file paths, commands, IDs, error text, numbers with more than
  two significant figures, or code in the digest itself. Those belong in the drill-down. A
  point should read like something you'd say out loud across a desk.
- **Nothing is lost, only deferred.** Every specific you strip out of a point must be
  recoverable when the user expands that point. If you can't hang a detail on any of the
  numbered items, that's a sign you're missing a point, not a reason to drop the detail.
- **Say what you did on their behalf.** Any choice you made without asking (picked an approach,
  skipped something, worked around an error) gets its own `[decision]` or `[changed]` item.
  These are the highest-regret things to bury.
- **No preamble, no apology.** Don't say "Sorry, that was a lot." Just produce the digest.

## Drill-down behavior

When the user replies with a number:

- Expand only that item. Lead with the one-sentence answer, then supporting bullets. Now the
  specifics come back: paths, values, error text, commands — in a code block if they are code.
- Keep it to roughly a screen (≈150 words) unless the content is inherently long, such as a
  diff or a log excerpt. If more exists, end with "More on this? (y)" rather than dumping it.
- Keep the original numbering stable for the rest of the exchange so "back to 4" always means
  the same thing. If expanding reveals sub-points, letter them (4a, 4b) so they can be
  drilled into too.
- After answering, restate the "Needs you" line if it is still open, then wait again.

When the user replies "go" (or otherwise gives the go-ahead), resume the task where you left off.

## Example

Prior turn: 40 lines of investigation across six tool calls, ending in a long explanation of a
Hightouch sync failure, a schema change made along the way, and two alternative fixes.

Digest:

```
**Bottom line:** The sync is failing on one bad row; I've narrowed it to a data issue, not config, and need you to pick a fix.

1. [decision] Two fixes possible: patch the one row, or add a filter so it never recurs.
2. [changed] I added a column to the staging model while investigating — harmless but real.
3. [found] The failure is a single account with a malformed country code.
4. [found] Every other row syncs fine; this has been failing for three days.
5. [risk] The filter approach hides future bad rows instead of surfacing them.

**Needs you:** Which fix — patch the row, or add the filter?

Reply with a number to expand, "all" for full detail, or "go" to continue.
```

---
name: confirm
description: Restate what the user is trying to accomplish and wait for their confirmation before executing anything.
disable-model-invocation: true
---

# Confirm

## Purpose

The user invokes this when they want to make sure you and they are aligned **before** you start doing work. The cost of a wrong turn — reading the wrong files, writing the wrong code, sending the wrong message — is far higher than the few seconds it takes to confirm intent up front. This is a deliberate checkpoint, not a formality.

## What to do

Before executing anything, reiterate back to the user what you understand they are trying to accomplish, so they can confirm or update your understanding. Then **stop and wait** — do not begin the task until they respond.

Structure the readback so it's easy to skim and correct:

1. **Goal** — the outcome they want, in one or two sentences, in your own words (not a verbatim echo of their prompt — restating it in your own words is what surfaces misunderstandings).
2. **What I'll do** — the concrete steps or approach you intend to take, briefly.
3. **Assumptions** — anything you're inferring that isn't spelled out, and any decision points where you picked a default. This is the highest-value part: it's where silent misunderstandings hide.
4. **Open questions** (only if any genuinely block you) — things you can't proceed sensibly without knowing.

Keep it tight. The point is a fast alignment check, not a plan document — a wall of text defeats the purpose. If the task is genuinely trivial and there's nothing to misunderstand, say so in a sentence rather than padding it out.

End by explicitly inviting correction — e.g., "Does that match what you're after, or should I adjust?" — and then wait for their reply before proceeding.

---
name: skillify-this
description: Turn a process into a reusable skill (SKILL.md), either one the user describes from scratch or one they just finished doing with the agent in this conversation. Detects which situation it is in. Invoked at the start of a chat, it interviews the user and builds the skill step by step with live validation. Invoked after completed work, it extracts the proven steps, corrections, and judgment calls from the conversation and generalizes them. Use for "skillify this", "make this a skill", "turn what we just did into a skill", "I want to be able to do this again", "save this process", "codify my weekly X process", or /skillify-this. Trigger even without the word "skill" when the user wants a repeatable version of a process.
---

# Skillify This

Turn a process into a reusable skill. There are two entry points, and they need different methods:

| | **Build mode** (fresh start) | **Capture mode** (just did it) |
|---|---|---|
| Situation | User has a process in mind but hasn't run it with you | User just completed the process with you in this conversation |
| Source of truth | The user's description | The conversation transcript: what actually happened |
| Main risk | Instructions that read well but fail on real tool calls | Overfitting the skill to this one run |
| Method | Build and run each step live, one at a time | Extract, generalize, then test on a fresh input |

## Step 0: Detect the mode

Look at the conversation that came before the invocation.

- **Capture**: substantial completed work precedes the invocation (tool calls, produced outputs, iterations or corrections), and the user is pointing back at it ("this", "what we just did").
- **Build**: invoked at or near the start of the chat, or the earlier conversation is unrelated to the process the user wants to skillify.
- **Hybrid**: the user is partway through a process and decides mid-stream to make it a skill. Capture the completed steps, then Build the remaining ones.
- If the earlier conversation contains more than one distinct process, ask which one.

Confirm the detected mode in one line before doing anything else, naming the process, for example: "Looks like you want to turn the renewal-report process we just ran into a skill. Right?" or "Starting fresh. Walk me through the process you have in mind." The detection is usually right, and this one line catches it when it isn't. Don't ask anything else yet.

## Shared principles

1. **Reuse first.** Check the user's available skills before proposing anything new. Reference an existing skill by its real invocation name instead of re-describing what it does.
2. **Strict modularity.** Create a separate sub-skill only if the user will invoke it on its own, outside this skill. Otherwise keep the instructions inline. Every unnecessary sub-skill adds triggering ambiguity and maintenance surface.
3. **Right-size the process.** A simple single-purpose task becomes one short SKILL.md, without step-by-step ceremony. Use the full protocol for multi-step workflows.
4. **Explain the why.** Write each instruction with its reason ("filter to active accounts, because churned accounts break the ARR join"). Reasons let a future agent handle cases the instructions didn't anticipate; bare commands don't.
5. **No secrets or personal data in the skill.** Tokens, passwords, and personal identifiers stay out of the files. Reference where to fetch them at runtime instead.
6. **Gate side effects.** Before any step that sends, posts, writes to, or deletes from an external system (email, Slack, CRM, shared docs), confirm with the user and prefer a safe target: a draft, a sandbox, or a message to themselves.

## Build mode

1. Ask for the workflow steps if the user hasn't already provided them.
2. Restate your understanding and flag critical gaps only: auth that may not exist, ambiguous data sources, steps with unclear success criteria. Skip superficial questions.
3. Once the user confirms, process the workflow one step at a time:
   1. **Check and classify**: does an existing skill already cover this step? If not, does it warrant a sub-skill (Principle 2)?
   2. **Build or draft**: write the step's instructions inline, or create the sub-skill. If a skill-creator skill is available in this environment, use it for sub-skills.
   3. **Test and refine**: run the step live and check that the tools, auth, and output quality hold up. Iterate on the user's feedback. Never mark a step done on the strength of a dry read; run it.
   4. **Stop.** Do not start the next step until the user explicitly approves.

Validating each step live is what makes the finished skill trustworthy: batch-drafted steps look plausible but fail on real tool calls (wrong tool names, missing auth, malformed output). Don't assemble the final skill until every step is tested and approved, because later steps often reshape earlier ones.

## Capture mode

The steps were already proven once, in this conversation. Your job is to distill and generalize them, not to rebuild them.

1. **Reconstruct from the transcript.** List the steps that actually worked, in order, with the exact tools, commands, queries, and parameters used. For each step, note its input, its action, its output, and how success was judged.
2. **Mine the corrections.** Every user correction ("no, use X instead", "that's wrong because…") and every failure that was then fixed becomes an explicit rule or gotcha in the skill. This is the highest-value content: it encodes exactly what the agent would get wrong next time.
3. **Drop the dead ends.** Leave abandoned approaches out of the steps. Keep the lesson as a one-line "don't do X, because Y" only if X would tempt a future agent.
4. **Parameterize.** List every concrete value from this run (customer names, dates, file paths, IDs, thresholds) and classify each one: an input asked for on every run, a fixed constant, or a value derived from other inputs. Show the user this table; they decide.
5. **Surface unstated judgment.** Wherever the user made a choice the transcript shows but doesn't explain (picked these rows, rejected that draft), ask for the rule behind it. Ask only about choices that would change the output of the next run.
6. **Flag gaps honestly.** If earlier parts of the conversation were summarized or truncated, or the user did a step outside the chat, say so and ask. Never fill a gap with a plausible-looking command you didn't see work.
7. **Review in one pass.** Present the reconstructed steps, the inputs table, and the rules together, and get approval. Unlike Build mode, don't approve steps one at a time, because they already ran successfully.
8. **Test generalization.** Offer a test run on a different input than the original. Never re-run against the original targets: that duplicates side effects (a second email, a second CRM write). If the user declines the test, say plainly that the skill has been validated only on the original run.

## Hybrid

Run Capture steps 1–7 on the completed portion, then switch to the Build loop for the remaining steps, then assemble.

## Final assembly

Write the finished skill:

- **Frontmatter**: a kebab-case `name`, and a `description` that states what the skill does and when to trigger it, including the phrases the user would naturally say. All "when to use" language belongs in the description, not the body. Keep it under 1KB.
- **Body**: purpose; inputs (from the parameter table); steps in validated order, with the exact tool names and commands that were proven live, not paraphrases; a rules-and-gotchas section (from corrections and dead ends); and what the output looks like when done.
- Move long reference material into a `reference/` file and point to it from the body.

**Install.** Confirm the location with the user. The default is the user's personal skills folder for the agent they're using; find it by checking where their other installed skills live (Claude Code, for example, uses `~/.claude/skills/<name>/SKILL.md`). If you can't write files in this environment, output the complete files for the user to save.

Afterwards, tell the user how to invoke the new skill, and that it may need a new session to load.

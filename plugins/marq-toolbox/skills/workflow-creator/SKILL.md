---
name: workflow-creator
description: Act as a Workflow Architect that converts the user's multi-step workflow into a master SKILL.md — processing one step at a time with live validation and explicit approval gates before anything is locked in. Use whenever the user wants to turn a repeatable process, routine, runbook, or multi-step workflow into a skill, e.g. "turn my weekly reporting process into a skill", "build a master skill for my X workflow", "codify this process so I can rerun it", or /workflow-creator. Trigger even if they don't say the word "skill" but describe wanting a repeatable, automated version of a multi-step process. Do NOT use for creating a single, simple, self-contained skill — invoke skill-creator directly for that.
---

# Workflow Creator

Act as a **Workflow Architect**. Guide the user through building a master `SKILL.md` workflow by processing their input one step at a time.

The reason for the one-step-at-a-time discipline: a master workflow skill is only as good as its weakest step. Batch-drafting all steps at once produces instructions that *look* plausible but fail on real tool calls (wrong MCP tool names, missing auth, malformed output). Validating each step live, in-session, before locking it in is what makes the finished skill trustworthy.

## Core Principles

1. **Strict Modularity**: Create a standalone sub-skill ONLY if the user will explicitly invoke it on its own outside this master workflow. Be conservative — if it isn't useful on its own, keep the instructions inline within the master skill text. Every unnecessary sub-skill adds triggering ambiguity and maintenance surface.
2. **Reuse First**: Check the user's existing skill catalog (the available-skills list in context) before proposing anything new. Reuse an existing skill via its supported invocation mechanism; reference its `/` command in the master skill text when available.
3. **Live Validation**: Execute each step live during the session to verify tools (MCP servers, CLI, auth) and output quality before locking it in. Never mark a step "done" on the strength of a dry read — run it.

## Execution Protocol (Loop Per Step)

Process the workflow strictly one step at a time. Pause for the user's confirmation before proceeding to the next step.

1. **Check & Classify**: Check whether a matching skill already exists in the catalog. If not, decide whether a sub-skill is warranted based on standalone utility (Principle 1).
2. **Build or Draft**: Either write plain-text instructions for the master skill inline, OR invoke the skill-creator skill (via the Skill tool, name `skill-creator:skill-creator`) and fully create, test, validate, and finalize the sub-skill before continuing.
3. **Test & Refine**: Run the step live, surface performance or tool issues, and iterate based on the user's feedback.

**STOP after each step.** Do not begin the next step until the user explicitly approves proceeding. Do not begin creating the master `SKILL.md` until every component step has been tested and approved. This gate exists because later steps often reshape earlier ones — assembling early bakes in stale decisions.

## Final Assembly

Once all steps are verified and approved, assemble and output the complete master `SKILL.md`:

- Valid YAML frontmatter with `name` and a description that states both what the workflow does and when to trigger it (all "when to use" language belongs in the description, not the body).
- Steps in their validated order, with the exact tool names, commands, and invocation mechanisms that were proven live — not paraphrases.
- References to reused skills by their real installed names.
- Confirm with the user where to install it (default: global, `~/.claude/skills/<name>/SKILL.md`), then write it there.

## Immediate Next Steps (on invocation)

1. Ask the user for their workflow steps if they haven't already been provided in the invocation arguments or earlier conversation.
2. Reiterate your understanding of the workflow and flag any critical missing information (auth that may not exist, ambiguous data sources, steps with unclear success criteria). Do not ask superficial questions.
3. Once the user confirms your understanding, begin Step 1 of the Execution Protocol.

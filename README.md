# Marq Toolbox Plugins

Public plugin marketplace for Marq employees. It contains one plugin (`marq-toolbox`) with four general-purpose skills, packaged for both Claude Code and Codex.

The source is publicly readable for installation and inspection. It remains unlicensed; public availability does not grant permission to copy, modify, or redistribute it.

## Skills

- **tldr** — Cognitive-load circuit breaker: rewrites the last stretch of output as a 3–7 item numbered digest you can drill into one item at a time, then pauses until you choose.
- **confirm** — Restates the requested outcome and obtains confirmation before acting.
- **zoom-out** — Steps back from the immediate task to reframe the problem, surface the bigger goal, and check whether the current path still serves it.
- **workflow-creator** — Converts a repeatable multi-step workflow into a structured master skill, one validated step at a time.

## Prerequisites

None. All four skills are self-contained and need no connected apps.

## Install

### ChatGPT / Codex (Marq workspace)

Open the Plugins directory, select the **Marq** tab, and install **Marq Toolbox**. Updates arrive automatically after each daily sync from this repo.

### Claude Code

```
/plugin marketplace add marqHQ/marq-marketplace-toolbox
/plugin install marq-toolbox@marq-toolbox-plugins
```

### Codex CLI

```bash
codex plugin marketplace add https://github.com/marqHQ/marq-marketplace-toolbox
codex plugin add marq-toolbox@marq-toolbox-plugins
```

Registering the GitHub repository without a pinned ref lets Codex track marketplace updates from its default branch.

## Layout

```
.claude-plugin/marketplace.json    Claude Code marketplace manifest
.agents/plugins/marketplace.json   Codex marketplace manifest
plugins/marq-toolbox/              The plugin (skills/, both plugin manifests, assets)
```

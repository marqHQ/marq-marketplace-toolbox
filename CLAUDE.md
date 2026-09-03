# CLAUDE.md

Follow [AGENTS.md](AGENTS.md) — it is the canonical agent guide for this repo (layout, skill-editing rules, definition of done).

Claude Code specifics:

- This repo is itself a Claude Code plugin marketplace (`.claude-plugin/marketplace.json`); skills auto-discover from `plugins/marq-toolbox/skills/`.
- When testing skill changes locally: `/plugin marketplace add <path-to-this-repo>` then `/plugin install marq-toolbox@marq-toolbox-plugins`.

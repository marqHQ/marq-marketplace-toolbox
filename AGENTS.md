# Agent guide

Dual-compatible plugin marketplace (Claude Code + Codex) for general-purpose Marq skills. Human-facing overview and install steps: [README.md](README.md).

## Commands

No build step and no test suite. Validate manifests after editing them:

```
python -m json.tool .claude-plugin/marketplace.json
python -m json.tool .agents/plugins/marketplace.json
python -m json.tool plugins/marq-toolbox/.claude-plugin/plugin.json
python -m json.tool plugins/marq-toolbox/.codex-plugin/plugin.json
```

## Layout

- `plugins/marq-toolbox/skills/<name>/` — each skill: `SKILL.md`, optional `reference/` or `assets/`, and `agents/openai.yaml` (Codex interface metadata).
- Marketplace manifests: `.claude-plugin/marketplace.json` (Claude Code) and `.agents/plugins/marketplace.json` (Codex).
- Plugin manifests: `plugins/marq-toolbox/.claude-plugin/plugin.json` and `plugins/marq-toolbox/.codex-plugin/plugin.json`. Keep name, description, and version in sync across both when editing either.

## Rules — do not weaken when editing skills

- **This repo is public.** Never commit employee data (names with emails, Slack user IDs, rosters), credentials, tokens, OAuth client files, or anything machine-specific. Skills that need such data must fetch it at runtime from a marq.com-restricted location or build it per user on first run.
- Skill frontmatter `description` stays under 1KB (target 500–800 bytes) so Codex surfaces every skill. Put detail in the body, not the description.
- `version` in both plugin manifests is plain semver (no `+build` metadata). Bump it on every change that should reach installed users.
- `interface.defaultPrompt` in the Codex manifest holds at most 3 entries.
- Reference bundled files relative to the skill directory (`reference/x.md`, `assets/y.json`). Never use `${CLAUDE_PLUGIN_ROOT}` in skill prose.
- Skills must work for any Marq employee: no hardcoded personal paths, emails, or user IDs.

## Distribution

The Marq ChatGPT workspace imports this repo as a GitHub marketplace and re-syncs the default branch daily (or on **Sync now** by the importing admin). Pushing to `main` is the release step; members install the plugin once and receive updates automatically.

## Definition of done

- All four manifests parse as JSON and agree on plugin name and version.
- Every skill directory has `SKILL.md` with `name` + `description` frontmatter and `agents/openai.yaml`.
- Version bumped in both plugin manifests.
- README skill list matches `plugins/marq-toolbox/skills/`.

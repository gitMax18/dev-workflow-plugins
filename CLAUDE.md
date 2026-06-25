# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Local development commands

```shell
# Test a plugin in isolation (no install required)
claude --plugin-dir ./plugins/spec-driven

# Validate manifests and plugin structure
claude plugin validate .

# Reload all plugins in the current session without restarting
/reload-plugins
```

## Architecture

This repo is a **Claude Code plugin marketplace**. It has two layers:

- **Marketplace** (root) — `.claude-plugin/marketplace.json` catalogs all plugins. This is the entry point when a user runs `/plugin marketplace add gitMax18/dev-workflow-plugins`.
- **Plugins** (`plugins/<name>/`) — each plugin is self-contained with a `plugin.json` manifest and a `skills/` directory. Skills are auto-discovered from `skills/*/SKILL.md`.

### Plugin structure

```
plugins/<name>/
├── .claude-plugin/plugin.json   # manifest: name, version, description, keywords
├── README.md                    # user-facing docs
└── skills/<skill-name>/
    └── SKILL.md                 # YAML frontmatter + Markdown instructions
```

### SKILL.md frontmatter fields

| Field | Required | Notes |
|---|---|---|
| `name` | yes | matches the directory name |
| `description` | yes | primary triggering mechanism — used by Claude to decide when to invoke the skill |
| `disable-model-invocation` | no | set to `true` to restrict the skill to explicit command invocation only (prevents auto-triggering) |

### Version sync rule

`version` must be kept identical in both places whenever a plugin changes:
- `plugins/<name>/.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json` (in the corresponding plugin entry)

Follows [SemVer](https://semver.org/lang/fr/): bump patch for fixes, minor for new skills, major for breaking changes.

## Plugins

### `spec-driven`

A five-step spec-driven development cycle. Each skill is command-only (`disable-model-invocation: true`):

| Command | Output directory | Role |
|---|---|---|
| `/spec` | `_specs/` | Functional spec — the WHAT and WHY, never the HOW |
| `/plan` | `_plans/` | Technical plan derived from spec — runs in plan mode |
| `/implement` | (code) | Implements the plan, checks off spec tasks |
| `/review-workflow` | updates `_specs/`, deletes `_plans/` | Closes the cycle |
| `/front-contract` | `_front/` | Frontend integration contract from spec + backend code |

Task identifiers (`1`, `1-1`, `1-2`…) are stable across the full cycle — never renumbered, used as the traceability key from spec through to code.

### `versionning`

Single skill (`conventional-commit`) that auto-triggers on any commit request. Operates strictly on already-staged files — never runs `git add`.

## Adding a new skill to an existing plugin

1. Create `plugins/<plugin>/skills/<skill-name>/SKILL.md`
2. Bump the plugin version in `plugins/<plugin>/.claude-plugin/plugin.json`
3. Mirror the same version bump and update the description in `.claude-plugin/marketplace.json`
4. Update `plugins/<plugin>/README.md` (commands table + diagram)
5. Update the root `README.md` (plugins table + structure section)

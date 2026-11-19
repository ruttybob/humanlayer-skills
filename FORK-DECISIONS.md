# Fork decisions

One entry per local change to an upstream-owned path (`plugins/**`). Add the entry before the change lands; update it when the decision changes. An entry names the paths, the delta, why, and the sync policy — what a sync must do when upstream touches the same lines.

## Entries

### `disable-model-invocation: true` on user-invoked skills

- **Paths**: `plugins/design-control-loop`, `plugins/rpi`, `plugins/visual-pr` — each at `skills/<name>/SKILL.md`.
- **Delta**: one frontmatter line per file.
- **Why**: the flag removes the skill from the model entirely; a description-only hint would still ride in context on every turn.
- **Sync policy**: the flag must survive every merge. When a conflict touches those frontmatter blocks, take upstream content and re-add the flag. `scripts/check-sync-invariants.ts` fails without it.

`show-me` carries the same flag from upstream itself — no local delta, nothing to re-apply.

### pi-compatible frontmatter on rpi agents

- **Paths**: `plugins/rpi/agents/` — all five `*.md` agent definitions.
- **Delta**: `tools` lists renamed to the pi tool set (`read, grep, find, bash`, plus `edit, write` for the implementer); `Skill` and `TodoWrite` dropped (no pi equivalents in child sessions); `model:` lines removed so children inherit the session default; every agent lists `codemode` — the pi builtin registers it in native children straight from the allowlist. Keep the concrete tools in the allowlist: a `codemode`-only child binds an empty sandbox (the registry mirrors the session tools — verified by failed smoke runs), so codemode is additive, never a replacement. `web-search-researcher` also carries `mcp:brave-search` (the user-level `~/.pi/agent/mcp.json` server) for web search; an agent with `mcp:` tools must launch as a background child (async).
- **Why**: these files are symlinked into `~/.pi/agent/agents/` and loaded by pi-subagents, which validates tools and models in pi terms.
- **Sync policy**: keep pi tool names when upstream touches those lines; when upstream adds a tool to an agent, re-map it to the pi name instead of taking the line verbatim. Accepted trade-off: the Claude Code `agents/` loader now sees pi tool names — pi is the primary harness for this fork.

### Bumped action majors in the loop template

- **Paths**: `plugins/design-control-loop/skills/design-control-loop/references/workflow-template.yml` (moved there when cbd2577 folded build-iterated-agentic-loop into design-control-loop; the fold silently regressed the bump to `@v4` — re-applied)
- **Delta**: `actions/setup-node@v4` and `actions/upload-artifact@v4` bumped to `@v5` (node20 deprecation).
- **Why**: the template is the source for newly generated loops; every generated workflow inherited the deprecated actions.
- **Sync policy**: keep the bumps; adopt any higher major upstream ships.

Fork-only paths (`.github/**`, `.agents/**`, `pi/**`, `scripts/check-sync-invariants.ts`) have no upstream counterpart and are not decisions — upstream cannot conflict with them. One caveat: `pi/skills/<name>/` is the pi-facing mirror of every plugin skill — the single place `~/.pi/agent/skills/*` symlinks point into this repo. Adapted copies: `show-me` (every HTML artifact must ship a dark/light theme toggle, per its `references/theme-toggle.html`), `design-control-loop` and `build-iterated-agentic-loop` (pi runner template in `agent-runner-templates.md`, `$SKILLBASE` line, `.agents/skills/` preferred for actuator skills; BIAL also carries `disable-model-invocation`), and `rpi` (moved from `pi/rpi`; keeps the pi tool-mapping described above). `build-iterated-agentic-loop` is restored from upstream@4d69d20: locally cbd2577 folded the plugin into design-control-loop, but upstream still ships it and pi keeps it standalone. `visual-pr` and `narrow-react-prop-types` are byte-verbatim copies — refresh from `plugins/` on every sync (`improve-claude-md` is intentionally not mirrored). For adapted copies, re-derive from upstream and re-apply the delta instead of merging.
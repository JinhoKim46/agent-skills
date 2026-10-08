# Setup and plugin skills

Reached from [`SKILL.md`](SKILL.md) when `docs/agents/issue-tracker.md` is missing, or when a *read* skill has to be found.

## When the setup is missing

Report it in the announcement line as `setup: missing`, then by size:

- **Trivial or Small** → build. Add one line to the final report: "This repo has no `docs/agents/` setup; run it before the next Medium+ task (I can do it)."
- **Medium, Large or Fog, or the user named an issue** → run `setup-matt-pocock-skills` (*read*) first. It asks its own questions and writes `docs/agents/*` plus the `CLAUDE.md` block; land those as their own small PR (or in the work's first PR if the project allows), then continue the route.
- **The user declines the setup** → use GitHub Issues when `git remote -v` points at GitHub and `gh auth status` succeeds, else local Markdown under `.scratch/`, and tell every routed skill which tracker it is.

## Finding a read skill

Use the plugin's copy, newest version:

```bash
ls -d ${CLAUDE_CONFIG_DIR:-~/.claude}/plugins/cache/mattpocock/mattpocock-skills/*/skills/*/<name>
```

When the plugin is not installed, fall back to the project's `.claude/skills/<name>/`, then `~/.agents/skills/<name>/`. A project copy wins over the plugin only when the project's `CLAUDE.md` says it customises that skill; otherwise a project copy is an older vendored one.

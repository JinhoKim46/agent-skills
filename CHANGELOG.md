# Changelog

## 1.2.0 (2026-10-08)

### route-work

- Optional phase log: when a project's `CLAUDE.md`, `AGENTS.md` or `docs/agents/` names one, every phase announcement is also recorded there, and a Large build's dispatcher records each ticket's start, phase changes, PR and close. Projects that name none see no change, and a failing log never stops the work.

## Marketplace (2026-10-08)

- `code-flow-report` is no longer listed in the `jinho` marketplace, so this repository covers route-work only. It is still available from [its own repository](https://github.com/JinhoKim46/code-flow-report).

## 1.1.0 (2026-10-08)

### route-work

- Announces the size, the signal that decided it, and only the next phase; each later phase is announced when it starts.
- A bug with an unknown cause is sized after diagnosis instead of before it.
- `SKILL.md` keeps what every route needs; issue handling (`ISSUES.md`), the Large ticket graph (`LARGE.md`) and missing setup (`SETUP.md`) are read only when the route reaches them.
- Shorter description; rules restated as the target behaviour; the Common mistakes table, which repeated rules from the body, is removed.

## 1.0.0 (2026-10-07)

- First release of route-work, packaged as a Claude Code plugin.

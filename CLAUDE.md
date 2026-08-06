# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working mode: orchestrator

Claude (Fable) acts as an orchestrator in this repository, not an executor. Decompose incoming work into well-scoped tasks and delegate them to subagents running on simpler models (via the Agent tool with a `model` override — `sonnet` for substantive writing/editing, `haiku` for mechanical or search tasks). Give each subagent a precise task statement with acceptance criteria, then verify its result before accepting it (e.g. review the diff, run `uv run mkdocs build --strict`). Do the work directly yourself only for trivial one-line changes where delegation is pure overhead.

## What this is

Personal portfolio / engineering-brand site for Igor Stureiko (https://stureiko.github.io), built with MkDocs + Material theme. All content lives as Markdown in `docs/`; site structure is defined by the `nav:` section of `mkdocs.yml`.

## Commands

Dependencies are managed with `uv` (Python >= 3.12, pinned via `.python-version`).

```bash
uv sync --locked          # install dependencies
uv run mkdocs serve       # local dev server with live reload (http://127.0.0.1:8000)
uv run mkdocs build --strict   # validate the site — this is what CI runs; fails on broken links/nav
```

There are no tests or linters; `mkdocs build --strict` is the validation gate.

## Deployment

Pushing to `main` triggers `.github/workflows/deploy.yml`, which runs `mkdocs build --strict` and then `mkdocs gh-deploy --force` to publish to GitHub Pages. Do not commit the generated `site/` directory (gitignored).

## Structure notes

- New pages must be added to `nav:` in `mkdocs.yml` or they won't appear in navigation; with `--strict`, pages missing from nav or broken internal links fail the build. Several planned architecture pages exist only as commented-out nav entries in `mkdocs.yml`.
- Some pages in `docs/architecture/` (e.g. `ai-agents.md`, `mlops-platform.md`) exist on disk but are not yet in the nav.
- Markdown extensions enabled include admonitions, superfences, tabbed content, footnotes, and emoji (Material style) — see `markdown_extensions` in `mkdocs.yml`.

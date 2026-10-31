# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working mode: orchestrator

Claude (Fable) acts as an orchestrator in this repository, not an executor. Decompose incoming work into well-scoped tasks and delegate them to subagents running on simpler models (via the Agent tool with a `model` override — `sonnet` for substantive writing/editing, `haiku` for mechanical or search tasks). Give each subagent a precise task statement with acceptance criteria, then verify its result before accepting it (e.g. review the diff, run `uv run mkdocs build --strict`). Do the work directly yourself only for trivial one-line changes where delegation is pure overhead.

## What this is

Personal portfolio / engineering-brand site for Igor Stureiko (https://stureiko.github.io), built with MkDocs + Material theme and published bilingually (English default, Russian) via `mkdocs-static-i18n`. All content lives as Markdown in `docs/`; site structure is defined by the `nav:` section of `mkdocs.yml`. There is no custom CSS/JS, no theme overrides, and no `extra:` config — everything is driven by `mkdocs.yml` and the Markdown files.

## Commands

Dependencies are managed with `uv` (Python >= 3.12, pinned via `.python-version`).

```bash
uv sync --locked               # install dependencies
uv run mkdocs serve            # local dev server with live reload (http://127.0.0.1:8000)
uv run mkdocs build --strict   # validate the site — this is what CI runs
```

There are no tests or linters; `mkdocs build --strict` is the validation gate. The build prints a long banner from the Material team about MkDocs 2.0 — it is informational, not a failure. `main.py` at the repo root is a `uv init` leftover and is unrelated to the site.

## Deployment

Pushing to `main` triggers `.github/workflows/deploy.yml`, which runs `mkdocs build --strict` and then `mkdocs gh-deploy --force` to publish to GitHub Pages. Do not commit the generated `site/` directory (gitignored).

## Navigation and strict mode

- A page appears on the site only if it is listed in `nav:` in `mkdocs.yml`. Pages on disk that are missing from `nav:` do **not** fail `--strict`; MkDocs only logs them at INFO level, so they build silently and are unreachable. Currently orphaned this way: `docs/architecture/ai-agents.md`, `docs/architecture/mlops-platform.md`, `docs/articles/From_model_to_system/ML_System_Design_outline.md` (a working outline, not a publishable page), and `docs/articles/Team_Agent_Context_Platform/Team_agent_context_platform.md`.
- What `--strict` does fail on: a `nav:` entry pointing to a non-existent file, and broken internal links or anchors in Markdown. Check both when renaming or moving pages.
- Several planned architecture pages exist only as commented-out titles (without paths) under `Architecture:` in `nav:`.

## Bilingual content (mkdocs-static-i18n)

- `docs_structure: suffix`: a Russian translation is `<name>.ru.md` placed next to its English `<name>.md`. There is no `ru/` folder.
- `fallback_to_default: true`: pages without a `.ru.md` file still appear in the Russian site, showing the English content. Only two articles are translated so far (`three_questions` and `ml_system_design`); all other pages fall back.
- Russian navigation labels come from `plugins.i18n.languages[ru].nav_translations` in `mkdocs.yml`, keyed by the exact English nav title. When adding or renaming a `nav:` entry, add/update the matching key there, otherwise the Russian nav shows the English title.
- The build runs twice (`en` to `site/`, `ru` to `site/ru/`); both passes must be clean.

## Content conventions

- Articles live in `docs/articles/<Topic_Dir>/<slug>.md` (+ optional `<slug>.ru.md`) and are listed on the overview page `docs/articles.md` as cards. When adding an article: create the files, add the nav entry and its `nav_translations` key, and add a card on `docs/articles.md` whose link matches the nav path.
- Projects live in `docs/projects/*.md` with an overview at `docs/projects/index.md`; the overview uses card-style links that must stay in sync with the nav.
- The downloadable resume PDF is `docs/assets/documents/Igor_Stureiko_AI_Architect.pdf`, linked from `docs/resume.md` and `docs/contact.md`. Card grids (`<div class="grid cards" markdown>`) are used on the home, projects and articles overview pages.
- Markdown extensions enabled include admonitions, `pymdownx.details`, superfences, tabbed content (`alternate_style`), footnotes, task lists, `attr_list` + `md_in_html` (used for card grids), and emoji (Material twemoji) — see `markdown_extensions` in `mkdocs.yml`.

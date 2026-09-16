# AGENTS.md

Personal blog (Chinese content) built as a Zensical static site. Content-only repo: no Python code, no tests, no lint/typecheck — don't go looking for them.

## Commands

- Dev server: `uv run zensical serve` (Python 3.13 pinned via `.python-version`; deps in `pyproject.toml`/`uv.lock`, managed by uv)
- Production build: `uv run zensical build --clean` (output to `site/`, gitignored)

## Config gotcha

Site config lives in `zensical.toml` at the repo root — there is NO `mkdocs.yml`. Zensical is MkDocs-inspired but not drop-in compatible; check https://zensical.org docs before adding MkDocs-specific plugins/config. `nav` is commented out, so navigation is implicit, derived from the `docs/` directory structure.

## Content conventions

- All posts are Markdown under `docs/`; top-level Chinese-named dirs are the nav categories: `AI`, `大数据`, `生活知识`, `软件开发`. New posts go in the matching dir (create a new top-level dir to add a category).
- `docs/index.md` is the homepage.
- Posts start with YAML frontmatter `tags: ["...", "..."]` (all 44 existing posts do). Keep this for new posts.
- Filenames use Chinese with underscores instead of spaces (e.g. `pythonGUI实现大模型流式对话_whisper_ollama.md`). Filenames become URLs — avoid slashes/spaces; moving or renaming a post breaks its URL.
- Docs are pure Markdown, no local image/asset files currently; link images externally.

## Deploy

Pushing to `main`/`master` triggers `.github/workflows/docs.yml`, which `pip install zensical && zensical build --clean` and publishes `site/` to GitHub Pages at `https://qianfuxin.github.io/blog` (the configured `site_url`). Use relative intra-site links so they resolve under the `/blog` base path. History shows direct pushes to the default branch (no PR flow).

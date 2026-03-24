# AGENTS.md

## Big picture
- This repo is a Hugo site that builds a Reveal.js deck through the vendored `reveal-hugo` theme (`config.toml`: `theme = "reveal-hugo"`, `outputFormats.Reveal`, `publishDir = "build"`).
- The presentation source of truth is `content/_generator.md`; `shared-slides/preprocess.rb` regenerates `content/_index.md` from every `**/content/**/*generator.md` file. Treat `_index.md` as generated output.
- The Ruby preprocessor also expands `<!-- write-here "path" --> ... <!-- end-write -->` blocks by replacing them with file contents, optionally omitting the warning via `write-here-code`.
- Final site output lives in `build/`; compiled/intermediate Hugo artifacts live under `resources/_gen/`. Do not hand-edit either.
- `shared-slides/` and `themes/reveal-hugo/` are git submodules (`.gitmodules`); only edit them when the task explicitly targets shared tooling or theme internals.

## Where to change things
- Slide content: edit `content/_generator.md`, not `content/_index.md`.
- Theme/look-and-feel: edit `assets/custom-theme.scss`; Hugo compiles it because `params.reveal_hugo.custom_theme = "custom-theme.scss"` and `custom_theme_compile = true`.
- Repo-specific presentation helpers live in `layouts/shortcodes/` and `layouts/partials/reveal-hugo/`, not in the theme submodule.
- Static slide assets are referenced relative to `content/` (example: `![...](target%20system.png)` in `content/_generator.md`), so keep asset paths aligned with the existing `content/` files.

## Project-specific patterns
- Slides are reveal-hugo markdown with `---` separators and shortcode-heavy layout. Example: `content/_generator.md` uses `{{% multicol %}}{{% col %}} ... {{% /col %}}{{% /multicol %}}` for side-by-side content.
- Reuse the existing shortcodes before adding raw HTML:
  - `multicol.html` + `col.html` for Bootstrap-like columns
  - `code.html` and `import.html` for including file snippets or rendered markdown ranges
  - `tick.html` / `cross.html` for comparison tables already used in `content/_generator.md`
- `layouts/partials/reveal-hugo/head.html` injects Bootstrap, blur CSS, Font Awesome, and QR code support; `body.html` injects MathJax and print-PDF handling for YouTube embeds.
- Rich media is embedded directly in markdown (`<video ... data-src=...>`, `<iframe ...>` in `content/_generator.md`); this works because `markup.goldmark.renderer.unsafe = true` in `config.toml`.

## Build, preview, and deploy
- Linux local preview: `./shared-slides/serve.sh`
- Containerized preview: `UID=$(id -u) GID=$(id -g) docker compose up --build`
- One-off build flow (matches CI): `shared-slides/preprocess.rb && hugo`
- `shared-slides/serve.sh` reruns preprocessing on `content/` and `shared-slides/` changes via `inotifywait`, excluding generated `_index.md` files.
- CI in `.github/workflows/build-and-deploy.yml` sets up extended Hugo, runs the Ruby preprocessor, builds with `hugo`, inlines Mermaid via `cric96/inline-mermaid`, and deploys `build/` to `gh-pages`.

## Agent guardrails
- Prefer minimal edits in `content/_generator.md`, `assets/custom-theme.scss`, or `layouts/*`.
- Avoid editing `build/`, `resources/_gen/`, `content/_index.md`, or vendored files under `themes/reveal-hugo/` unless the task is explicitly about generated output or the submodule itself.
- If a visual change does not appear, check whether it belongs to generated markdown, compiled SCSS, or a theme partial override before changing the theme submodule.



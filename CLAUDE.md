# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page personal resume site for Tommy Tang, published at https://ttang.dev. It is a Jekyll site built by GitHub Pages' built-in Jekyll pipeline (there is no deploy workflow — pushing to `main` publishes). The custom domain is set by `CNAME`.

## Commands

```bash
bundle install                        # first-time setup
bundle exec jekyll serve --livereload # local preview at http://127.0.0.1:4000
JEKYLL_ENV=production bundle exec jekyll build   # build to ./_site as CI does

gem install html-proofer              # link/HTML validation, same as CI
htmlproofer ./_site --disable-external --allow-hash-href --ignore-empty-alt
```

CI (`.github/workflows/validate.yml`) runs on PRs to `main` only and does exactly the build + html-proofer steps above on Ruby 3.3. There are no tests.

### Windows local-dev notes

Verified working on Ruby 3.3.11 (x64-mingw-ucrt), installed via
`winget install RubyInstallerTeam.RubyWithDevKit.3.3`.

- `wdm` must stay at `~> 0.2`. Version 0.1.x cannot compile on Ruby 3.x — it calls
  `rb_thread_call_without_gvl` without the required header. It is declared with
  `platforms:` rather than `if Gem.win_platform?` so the Gemfile evaluates identically
  on Windows and on CI's Linux runner, keeping `Gemfile.lock` portable. If you regenerate
  the lock on Windows, re-run `bundle lock --add-platform x86_64-linux` or CI will fail.
- `html-proofer` needs `libcurl`, which Windows lacks. MSYS2's bundled keyring is stale
  and can't install it without elevation; the working route is
  `winget install cURL.cURL`, then copy that package's `libcurl-x64.dll` to
  `C:\Ruby33-x64\bin\libcurl.dll` (FFI looks for the unsuffixed name).
- `jekyll serve --detach` fails on Windows (`fork()` is unimplemented). Run it in the
  foreground, or as a background process without `--detach`.

## Architecture

Almost all content and layout lives in `_config.yml`, not in HTML/Markdown files. The site uses `remote_theme: sproogen/resume-theme`, which renders the entire page from the config's data structure:

- Top-level keys (`name`, `title`, `about_title`, `about_content`, `about_profile_image`, `og_image`, `darkmode`, `github_username`, `linkedin_username`) drive the header, about block, and social icons.
- `content:` is an ordered list of sections (`Client Engagements`, `Experience`, `Education`, `Certifications`). Each section has a `title` and `layout: list|text`; each item under it uses the theme's item keys — `layout: left|right`, `title`, `link`, `sub_title`, `caption`, `quote`, `description`. `description` is Markdown, so links inside it use Markdown syntax.
- Editing the resume means editing `_config.yml`. Jekyll does **not** hot-reload `_config.yml` changes — restart `jekyll serve` after editing it.

### Theme overrides

Local files take precedence over the remote theme, and two overrides exist because the
theme's own files are wrong for this site. Keep them when touching layout:

- **`_layouts/error.html`** — the theme's `_layouts/default.html` renders the resume
  sections directly and **never outputs `{{ content }}`**, so any page using it silently
  discards its body. Standalone pages such as `404.html` must use this layout instead;
  otherwise the 404 page renders as a byte-for-byte copy of the homepage.
- **`_includes/footer.html`** — the theme emits `<a href="mailto:{{ site.email }}">`
  unconditionally. `email` is intentionally unset in `_config.yml`, which produced a dead
  `mailto:` with no address. That single defect failed the html-proofer CI step on every
  run in the repo's history. The override only renders the link when `email` is set.

Everything else is thin:

- `index.md` — empty page with `layout: default`; the theme layout does the work.
- `assets/main.scss` — imports the theme (`@import 'modern-resume-theme'` — that is the theme's internal scss name, not a typo) and adds a few overrides. Add custom CSS here rather than vendoring theme files.
- `404.html`, `robots.txt` — both carry front matter so Liquid runs; `robots.txt` derives its sitemap URL from `url:` via `absolute_url`.
- `images/` — profile avatar (`devops.png`, also the `image:` social card) and favicon.

Because GitHub Pages builds this with its own gem set, only allowlisted plugins work. Current plugins are `jekyll-seo-tag` and `jekyll-sitemap`; adding an arbitrary gem will not take effect in production. Note this also means CI's `bundle exec jekyll build` is **not** the build that ships — production is Pages' built-in pipeline.

Two config keys are load-bearing in non-obvious ways:

- **`baseurl: ""`** must stay set explicitly. The `github-pages` gem otherwise infers
  `/pages/<owner>/<repo>` on unauthenticated local builds, which breaks every asset URL
  and anything using `absolute_url`. This was the cause of 4 of the 6 historical CI
  failures.
- **`defaults:` → `image`** is how the social card is set. `jekyll-seo-tag` reads `image`
  from *page* front matter, not site config, so setting `image:` (or `og_image:`) at the
  top level emits no `og:image` at all. With the default applied, seo-tag also upgrades
  `twitter:card` to `summary_large_image` automatically.

SEO metadata otherwise comes from `jekyll-seo-tag` via the theme's `{% seo title=false %}`,
which reads `site.description`. `devops.png` is 256×256 — adequate but undersized for a
social card, where 1200×630 is the target.

## Conventions

- Branch names follow `ftr/<topic>` for features and `fix/<topic>` for fixes; changes reach `main` via PR (see recent history). `@ttangwork` is the CODEOWNER.
- Commit subjects use conventional-style prefixes with the touched area, e.g. `fix(configs): ...`, `feat(images): ...`.
- Dates and certification expiries in `_config.yml` are hand-maintained prose — when updating one entry, check neighbouring entries for stale dates.

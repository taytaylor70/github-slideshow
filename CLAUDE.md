# CLAUDE.md

## Project Overview

**github-slideshow** is a GitHub Learning Lab training repository that teaches Git and GitHub fundamentals. It is a Jekyll-based static site that renders a reveal.js slideshow, deployed to GitHub Pages.

The slideshow is built by Jekyll: each Markdown file in `_posts/` becomes one slide, and reveal.js handles the browser-side presentation.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Static site generator | [Jekyll](https://jekyllrb.com/) via `github-pages` gem |
| Presentation framework | [reveal.js](https://revealjs.com/) (vendored in `node_modules/reveal.js`) |
| Templating | Liquid (Jekyll) |
| Syntax highlighting | Rouge (server-side) + Monokai theme (client-side) |
| Markdown parser | Kramdown |
| Emoji support | `jemoji` Jekyll plugin |
| HTML validation | `html-proofer` |

---

## Repository Structure

```
.
├── _config.yml          # Jekyll and reveal.js configuration
├── _posts/              # Slide content — one file per slide
│   └── 0000-01-01-intro.md
├── _layouts/
│   ├── presentation.html  # Root layout: wraps all slides in reveal's .slides div
│   ├── slide.html         # Single-slide layout (standalone view)
│   └── print.html         # Print/PDF layout
├── _includes/
│   ├── head.html          # <head> with reveal.js CSS links
│   ├── slide.html         # Renders one _posts entry as a <section> element
│   └── script.html        # reveal.js initialization script
├── index.html             # Entry point — iterates _posts and includes slide.html
├── node_modules/reveal.js # reveal.js library (committed/npm-installed)
├── package-lock.json
├── Gemfile / Gemfile.lock # Ruby gem dependencies
└── script/
    ├── setup              # Install Ruby/gem deps and init git submodules
    ├── server             # Run local Jekyll dev server
    ├── cibuild            # CI: build + html-proofer validation
    └── stage              # Deploy built site to internal GHE staging
```

---

## Development Workflows

### Setup

```sh
script/setup
```

Installs gem dependencies (`bundle install`) and initializes git submodules.

### Local Development Server

```sh
script/server
# equivalent to: bundle exec jekyll serve
```

Serves the site at `http://localhost:4000`. Live-reloads on file changes.

### CI Build (validates HTML)

```sh
script/cibuild
# equivalent to: bundle exec jekyll build --baseurl "." && htmlproofer _site/index.html --empty-alt-ignore
```

This is what runs in CI. A passing build requires both Jekyll compilation and HTML validation to succeed.

### Staging Deploy

```sh
script/stage [repo-name]
```

Builds the site with a staging baseurl and force-pushes `_site/` to a GHE staging instance's `gh-pages` branch.

---

## Adding or Editing Slides

Slides live in `_posts/` as Markdown files with Jekyll front matter. The filename date controls slide order — posts are iterated in **reverse chronological order** by `index.html`, so later dates appear first.

### Slide filename convention

```
YYYY-MM-DD-slide-title.md
```

### Minimal slide front matter

```yaml
---
layout: slide
title: "Slide Title"
---

Slide body content here (Markdown).
```

### Advanced front matter options

```yaml
---
layout: slide
title: "Custom Slide"
slide-id: my-anchor        # Sets id="" on the <section> element
classes:                   # Additional CSS classes on the <section>
  - my-class
data:                      # Sets data-* attributes for reveal.js (e.g. data-background)
  background: "#ff0000"
---
```

The `_includes/slide.html` template maps these front matter fields to reveal.js `<section>` attributes.

---

## Key Configuration (`_config.yml`)

- **Timezone:** `Europe/Berlin`
- **Permalink:** `/:title`
- **Theme:** Solarized dark (`solarized.theme: dark`)
- **Slide numbers:** `c/t` format (current/total)
- **Reveal transition:** `linear` with `slide` background transitions
- **Viewport:** 1000×920px, margin 0.1, scale range 0.2–1.5
- **`baseurl`** is commented out by default; uncomment and set (e.g. `/github-slideshow`) when deploying to a sub-path

To change a reveal.js option, edit the `reveal:` block in `_config.yml`. All keys map directly to `Reveal.initialize()` options.

---

## Formatting Conventions (`.editorconfig`)

| File type | Indent | Line ending |
|---|---|---|
| Default (`.sh`, etc.) | Tabs, width 4 | LF |
| `.json`, `.js`, `.css`, `.scss`, `.yml`, `.html` | Spaces, width 2 | LF |
| `.md`, `.markdown` | Spaces, width 4 | LF, final newline required |

- Trailing whitespace is trimmed on all files except Markdown.
- Editors with EditorConfig support enforce these automatically.

---

## Branching & GitHub Flow

- Default branch: `main`
- Development branches: feature branches merged via pull requests
- The repo is designed as a GitHub Learning Lab exercise — commits and PRs to forks are used as teaching checkpoints

---

## Deployment

The site is published to GitHub Pages from the `main` branch. Jekyll builds it automatically on push. No manual build step is needed for production — GitHub Pages runs `jekyll build` server-side.

If `baseurl` is needed (sub-path deployment), uncomment and set it in `_config.yml`.

---

## Dependencies

### Ruby (Gemfile)

- `github-pages >= 207` — pins Jekyll and all GH Pages plugins
- `html-proofer >= 3.13.0` — HTML link/image validation for CI
- `tzinfo-data` — Windows timezone support

### Node (package-lock.json)

- `reveal.js` — presentation framework, source served directly from `node_modules/reveal.js/`

Do not upgrade `github-pages` independently; it pins Jekyll and plugin versions to match GitHub Pages infrastructure.

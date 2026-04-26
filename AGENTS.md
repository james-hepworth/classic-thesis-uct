# AGENTS.md — `uct-classic-thesis` (Typst Universe package)

This file gives AI coding agents a working map of the repository. The repository is structured as a **Typst Universe template package** (`@preview/uct-classic-thesis:0.1.0`): the root is the package root, and `template/` holds the project skeleton that `typst init` copies into a new user project.

## Project Overview

A two-sided A4 PhD thesis / research-proposal template at the University of Cape Town in the Classic Thesis tradition. Mirrored margins, EB Garamond typography in spaced small caps, side-caption figures and tables, margin-set equation numbers, and a centred-column front matter (title page through acronyms) that switches to the asymmetric major-column layout in the main matter.

Migrated from a LaTeX implementation (now deleted) to native Typst.

## Technology Stack

- **Typesetter**: [Typst](https://typst.app/) ≥ 0.11 (uses `context`, `query`, `metadata`, `here()`).
- **Language**: Typst markup (`.typ`), TOML manifest.
- **License**: GPL v2 or later (see `LICENSE`).
- **No package manager**: dependencies are limited to fonts (see below) and the package itself when published.

## Font Dependencies

- **EB Garamond 12** — body
- **EB Garamond SC 12** — small caps and headings
- **New Computer Modern** — monospace

Bundled binaries live in `fonts/` for local development convenience but are excluded from the published package via `typst.toml` (see `exclude`).

## Repository Layout

```
.
├── typst.toml                 # Package manifest (Universe metadata, exclude rules, [template])
├── README.md                  # Universe-facing documentation (don't add an H1; Universe injects one)
├── LICENSE                    # GPL v2 text
├── lib.typ                    # Package entry point — re-exports from classicthesis-uct.typ
├── classicthesis-uct.typ      # Layout module: page setup, fonts, helpers, chapter/figure/equation blocks
├── thumbnail.png              # Universe thumbnail (page 1 of template @ 250 ppi)
├── assets/
│   └── UCT_logo.png           # Institutional logo bundled with the package (used by title-page by default)
├── template/                  # Files copied into a user's project by `typst init`
│   ├── main.typ               # Imports @preview/uct-classic-thesis:0.1.0, builds the document
│   ├── references.bib         # Example bibliography
│   ├── chapters/chapter01..07.typ  # Each exports `#let content = [...]`
│   └── graphics/              # Example figures
├── fonts/                     # Local font bundle (excluded from published package)
└── tmp/                       # Render artefacts (gitignored)
```

## Manifest (`typst.toml`)

- `name = "uct-classic-thesis"`, `version = "0.1.0"`, `entrypoint = "lib.typ"`.
- `[template]` section: `path = "template"`, `entrypoint = "main.typ"`, `thumbnail = "thumbnail.png"`.
- `exclude` keeps `fonts/`, `tmp/`, this `AGENTS.md`, and any stray PDFs out of the published bundle.

When bumping the version: update `version` in `typst.toml`, the import strings inside `template/main.typ` and `template/chapters/chapter*.typ`, and every `0.1.0` reference in `README.md`.

## Architecture

### Package entry (`lib.typ`)

Thin wrapper that does `#import "classicthesis-uct.typ": *` so downstream documents need only one import line. Add or remove re-exports here if the public surface should diverge from the layout module.

### Layout module (`classicthesis-uct.typ`)

Top-of-file constants double as the design tokens:

- Fonts: `body-font`, `display-font`, `heading-font`, `smallcaps-font`, `mono-font`.
- Colours: `accent`, `chapter-gray`, `label-color`, `rule-stroke`.
- Geometry: `inner-margin = 26mm`, `outer-margin = 50mm`, `page-top-margin = 32mm`, `page-bottom-margin = 27mm`, `front-matter-side-margin = (210mm − major-column) / 2`, `major-column = 130mm`, `caption-gutter = 6mm`, `caption-width = 30mm`.

Public functions (all re-exported by `lib.typ`):

| Symbol | Purpose |
|--------|---------|
| `configure(meta, body)` | Show-rule template; **must be invoked as `#show: configure.with(meta)`** so set rules reach the whole document. |
| `title-page(meta)`, `title-back(meta)` | Title recto and verso. The recto uses `meta.logo` when provided, otherwise it loads `assets/UCT_logo.png` from inside the package. |
| `abstract-page`, `contents-page`, `list-of-figures-page`, `list-of-tables-page`, `acronym-page` | Front-matter sections. Each emits a hidden `heading(level: 1, outlined: false, numbering: none)` via `front-heading` so the running-head logic and chapter-opener detection treat them like chapters without polluting the outline or chapter counter. |
| `chapter(title, number, body)` | Chapter opener: hidden level-1 heading (visible counterpart in `spaced_caps` + rule), large grey numeral in the outer margin via `place`. |
| `image_figure`, `side_caption_table`, `numbered_equation` | Major-column blocks with marginalia. |
| `bibliography-page(body)` | Heading + major-column block. **Takes content, not a path** — the user calls `#bibliography(...)` themselves so paths resolve relative to the project, not the package. |
| `major_column_block(body)` | Full-width block in the current text area. |
| `minor_top`, `minor_middle`, `minor_bottom` | Place content in the outer margin, with parity-aware alignment. |
| `folio_footer(format, shift: ...)` | Footer folio. Default `shift` is the asymmetric main-matter offset; pass `shift: 0mm` for symmetric front matter. |
| `classic_header()` | Running head; suppressed automatically on chapter-opening pages. |
| `running-head-title()`, `page_is_odd()` | Helpers. |

### Template (`template/main.typ`)

- Imports the package: `#import "@preview/uct-classic-thesis:0.1.0": *`.
- Defines `meta` (title, degree, name, supervisor, co-supervisor, faculty, department, university, location, date, funder).
- Applies `#show: configure.with(meta)`; sets heading numbering, citation form, and IEEE bibliography style.
- Front matter uses a centred single column (`front-matter-side-margin`, `header: none`, `folio_footer("i", shift: 0mm)`).
- Main matter switches to mirrored margins, restores `classic_header()`, uses `folio_footer("1")`.
- Chapters are fenced by `#pagebreak(to: "odd")` so each starts on a recto.

### Chapter files

Each `template/chapters/chapter0N.typ`:

```typst
#import "@preview/uct-classic-thesis:0.1.0": *

#let content = [
  == Section
  Body...
]
```

Use `==` / `===` / `====` for sections, sub-sections, and sub-sub-sections. **Do not use `=` (level 1)** — `#chapter()` already emits the hidden level-1 heading.

## Build & Test

There are no automated tests. Validate by compiling and visually inspecting the PDF.

### Local development

The `template/` files use the absolute package import (required by Universe), so direct `typst compile template/main.typ` only works once the package is reachable through the local Typst data directory:

```sh
# macOS
ln -snf "$(pwd)" "$HOME/Library/Application Support/typst/packages/preview/uct-classic-thesis/0.1.0"

# Linux
ln -snf "$(pwd)" "${XDG_DATA_HOME:-$HOME/.local/share}/typst/packages/preview/uct-classic-thesis/0.1.0"
```

Then:

```sh
typst compile template/main.typ template/thesis.pdf      # compile in place
typst init @preview/uct-classic-thesis:0.1.0 /tmp/test    # exercise the init flow end-to-end
```

### Regenerating the thumbnail

After any layout-affecting change:

```sh
typst compile -f png --pages 1 --ppi 250 template/main.typ thumbnail.png
```

Universe requires ≥1080 px on the long edge and ≤3 MiB total; `oxipng` is recommended if the file grows. The thumbnail is automatically excluded from the published bundle (it is referenced in `typst.toml`).

## Submission Notes (Universe)

- README must NOT contain an H1 — Universe renders the package name as the page heading.
- All template files must use the absolute package import (`@preview/...`), not relative file imports.
- Keep `typst.toml` and the import strings inside `template/` in sync on every version bump.
- Don't reference `thumbnail.png` from inside the package; it is for Universe display only.

## Important Agent Notes

- **Don't change the import path inside `template/` to a relative path** when troubleshooting locally — symlink the package instead.
- **Preserve mirrored-layout semantics.** Any margin, column, header, or caption change must respect `page_is_odd()` and the `inside`/`outside` distinction. Use `tmp/page-check/*.png` renders to verify.
- **Don't add level-1 headings in chapter files.** `#chapter()` and `front-heading` are the only places that emit hidden level-1 headings; introducing more breaks the running-head and chapter-opener-detection logic.
- **`bibliography-page` takes content, not a path.** A path passed in would be resolved relative to the package directory, not the user's project.
- **Geometry constants are shared.** Changing one (e.g. `outer-margin`) implicitly affects the folio-centring shift in `folio_footer` and the `front-matter-side-margin` derivation. Recompile and visually verify after any tweak.
- **British English** (`lang: "gb"`) — keep UK spelling in any sample prose.

# databyte #

Source for [davidsommers.com](https://davidsommers.com), built with Jekyll and
published by GitHub Pages.

## Setup

Ruby is pinned in [`.tool-versions`](.tool-versions) to match the version
GitHub Pages builds with. With [mise](https://mise.jdx.dev) or
[asdf](https://asdf-vm.com) installed, `cd` into this repo and:

````
mise install          # or: asdf install
bundle install
bundle exec jekyll serve
````

Then open <http://localhost:4000>. Edits to `_posts/`, `_layouts/`, `css/`, and
`index.html` rebuild automatically.

The pin is load-bearing: `github-pages` depends on `commonmarker`, which
requires Ruby `< 4.0`. On a newer Ruby, `bundle install` fails to resolve at
all. Use `bundle exec` rather than a bare `jekyll`, so you get the pinned
version rather than whatever is on `PATH`.

## Layout

| Path | What |
|---|---|
| `_posts/` | Posts. `YYYY-MM-DD-slug.md` (or `.html` for the older ones). |
| `_layouts/` | `default.html` is the shell; `post.html` wraps a single post. |
| `css/main.css` | The whole stylesheet, including both themes and syntax highlighting. |
| `index.html` | The archive, grouped by year. |

## Theming

Light and dark are both defined in `css/main.css` as custom properties. Dark
follows the reader's OS by default; the header toggle overrides it and stores
the choice in `localStorage`. An inline script in `_layouts/default.html` applies
the stored theme before first paint — keep it inline and above the stylesheet, or
dark-mode readers get a white flash on every page load.

## Links

* [Jekyll](https://jekyllrb.com/)
* [GitHub Pages dependency versions](https://pages.github.com/versions.json) —
  the source of truth for the Ruby pin.

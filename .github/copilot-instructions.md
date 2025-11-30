## Quick orientation

- **Project type:** Jekyll site (static site) serving fencing articles and TMX source files.
- **Where content lives:** posts in `_posts/`, TMX source files in `assets/sources/`, layouts in `_layouts/`.
- **Important config:** see `_config.yml` — `baseurl: "/articles"` and `permalink: /:year/:month/:day/:title/` (baseurl must match the repo path).

## What the codebase expects (practical patterns)

- TMX viewer: `tmx_viewer.html` is a client-side viewer that expects a query parameter `file` pointing at a TMX file under `assets/sources/`. Example URL: `/tmx_viewer/?file=1928-10-01-nadi-fioretto.tmx`.
- Viewer URL extras: supports `line` (numeric), `q` (search string), and `filter=1` (hide non-matches). Example: `?file=...&line=123&q=translat&filter=1`.
- TMX format: the viewer parses TMX as XML and uses `<tuv xml:lang="xx">` and `<seg>` nodes. Code assumes at least two `tuv` entries per `tu` for source/target assignment but falls back gracefully.
- Post-to-TMX naming convention: TMX source files use the same YYYY-MM-DD-title pattern as posts (e.g. `1928-10-01-nadi-fioretto.tmx`) so the viewer can construct a “Back to Article” link using the filename.
- Edit link: the viewer creates a top-level "✎ Request edit through GitHub" link that points to `assets/sources/<file>` using the repository owner/name/branch values declared in `tmx_viewer.html` (constants `GH_OWNER`, `GH_REPO`, `GH_BRANCH`).

## Build / run / test (how to be productive locally)

- The site is standard Jekyll. Typical local workflow on macOS (zsh):

  ```bash
  bundle install      # if you use Bundler / have a Gemfile
  bundle exec jekyll serve --livereload
  # then visit: http://127.0.0.1:4000/articles/tmx_viewer/?file=1928-10-01-nadi-fioretto.tmx
  ```

- Note: `_config.yml` sets `baseurl: "/articles"`. When testing locally include `/articles` in the served paths (Jekyll will apply `baseurl` automatically).

## Files and patterns to reference for change impact

- `tmx_viewer.html` — main viewer logic (client-side JS, DOMParser, fetch). Search/line behavior and Edit link live here.
- `_config.yml` — baseurl, permalink, markdown engine. Keep `baseurl` consistent with repo name.
- `assets/sources/` — TMX files. Adding a new TMX file here is sufficient for the viewer to load it by name.
- `_posts/` — article pages. Filenames follow `YYYY-MM-DD-title.md` and generate the article pages the viewer links back to.
- `_layouts/` — site chrome (header/footer). Use `{{ '/assets/sources/' | relative_url }}` or `{{ site.baseurl }}` to build correct asset links.

## Code conventions and small quirks to be aware of

- Liquid + client-side JS: The viewer mixes Liquid to compute `assetsPath`/`baseUrl` and client-side JS for runtime behavior. Avoid replacing runtime `fetch` URLs with server-side logic unless you also update the viewer's `assetsPath` construction.
- Baseurl duplication: `_config.yml` contains `baseurl` and `url` values; keep `baseurl` in sync with repo folder name so the smart back button URL resolves correctly.
- Minimal JS assumptions: the viewer sanitizes the `file` query parameter (`fileName.replace(/[^a-zA-Z0-9.\-_]/g, '')`) — keep filenames simple and POSIX-friendly.
- The viewer intentionally removed per-row edit anchors and uses a single top-level GitHub edit link — if you add row-level anchors update both rendering and the `GH_EDIT_BASE` logic.

## Example edit / PR flow (how to change a TMX and preview)

- Edit or add `assets/sources/1928-10-01-my-article.tmx` on a branch, push, then open the viewer URL to verify parse and UI:

  - `https://<your-gh-pages-host>/articles/tmx_viewer/?file=1928-10-01-my-article.tmx`

- Prefer opening the top-level edit link (the viewer constructs a GitHub editor link to `assets/sources/<file>`), not manual raw URLs — constants in `tmx_viewer.html` control owner/repo/branch.

## Helpful TODOs for AI agents working in this repo

- When changing `tmx_viewer.html`, run the viewer locally with `bundle exec jekyll serve` and test with at least one TMX in `assets/sources/`.
- If modifying permalink/baseurl behavior, update `_config.yml` and test the smart back button (it constructs post URLs from the TMX filename).
- Avoid touching `_layouts/` CSS without previewing posts, because styles affect readability of parallel translations.

---

If anything here is unclear or you'd like me to expand sections (for example, include exact Liquid snippets from `_layouts/default.html` or a short local verification script), tell me which part and I will iterate.

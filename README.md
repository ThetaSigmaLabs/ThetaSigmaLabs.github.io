# ThetaSigma Labs — Website

Source for **[thetasigma.org](https://thetasigma.org/)**, the home of ThetaSigma
Labs: open-source tools for astrophotography and 3D-print management, including
[`tsx_cmd`](https://thetasigma.org/tsx-cmd/) and lab notes from the home
observatory and print farm.

Built with [Hugo](https://gohugo.io/) and the
[Blowfish](https://github.com/nunocoracao/blowfish) theme. The site is a static build
deployed to GitHub Pages — there is no server, database, or backend.

## Prerequisites

| Tool | Version | Why |
| --- | --- | --- |
| Hugo | **extended** `0.161.1` | The non-extended build fails (Blowfish uses SCSS) |
| Go | `1.26.2` | The Blowfish theme is loaded as a Hugo Module and fetched via `go` |

```bash
brew install hugo go        # macOS; ensure `hugo version` reports "+extended"
```

## Local development

```bash
git clone git@github.com:ThetaSigmaLabs/ThetaSigmaLabs.github.io.git
cd ThetaSigmaLabs.github.io
hugo server -D              # serves http://localhost:1313 with drafts visible
```

The theme downloads automatically on first run (no submodules). To update it:

```bash
hugo mod get -u && hugo mod tidy
```

## Authoring content

```bash
hugo new lab-notes/my-post.md      # new lab note
hugo new content/<section>/page.md # new page in any section
```

New content is created with `draft = true` and uses **TOML** front matter
(`+++` fences). Drafts are excluded from production builds until that line is
removed. The top navigation is **not** automatic — add entries in
`config/_default/menus.en.toml`.

## Project layout

```
content/           Markdown pages (lab-notes/, tsx-cmd/, about, _index)
layouts/           Local overrides of Blowfish theme files (mirror the theme's path)
assets/css/        Custom CSS (custom.css — Blowfish auto-includes)
assets/img/        Site images (logo, hero, etc.)
archetypes/        Front-matter templates for `hugo new`
static/            Files copied verbatim, incl. CNAME (custom domain)
config/_default/   Split site config (params.toml, menus.en.toml, module.toml, etc.)
```

`public/` (build output) and `resources/_gen/` (asset cache) are generated and
git-ignored — never commit them.

## Deployment

Pushing to `main` triggers [`.github/workflows/hugo.yml`](.github/workflows/hugo.yml),
which builds the site and publishes it to GitHub Pages. The repository is
source-only; the rendered site is built fresh in CI on every push.

The custom domain is set via `static/CNAME` (→ `public/CNAME` on build). Repo
**Settings → Pages → Source** must be set to **GitHub Actions**.

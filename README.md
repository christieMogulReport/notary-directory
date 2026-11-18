# Everyday Directory

A tiny static directory site. Each page is a Markdown file with YAML front matter. A short Python build step turns those files into a fast, readable HTML site in `dist/`.

The niche is still being chosen, so names and copy in `config.yml` are placeholders. Change them when the topic is set. This repo is meant to live and build on this computer; it has not been deployed.

## Requirements

- Python 3.11+ (3.13 is fine)
- The packages in `requirements.txt` (`PyYAML` and `Markdown`)

`python3 build.py` and `python3 serve.py` create a local `.venv` and install those packages on first run.

## Commands

From `/workspace/directory-site`:

```bash
python3 build.py              # write the site into dist/ (draft pages are excluded)
python3 serve.py              # build, then preview at http://127.0.0.1:8000
python3 build.py --drafts     # include pages marked draft: true
python3 serve.py --drafts     # preview including drafts
rm -rf dist                   # delete a previous build
```

Set `PORT` or pass `--port` to `serve.py` to change the preview port.

If GNU Make is installed, `make build`, `make serve`, `make serve-drafts`, and `make clean` wrap the same commands.

## How to add a page

1. Create a new file in `content/pages/`. Use a lowercase, hyphenated name such as `content/pages/window-screens.md`.
2. Start the file with YAML front matter, then the page body in Markdown:

```markdown
---
title: Window screens in stock near you
slug: window-screens
description: A short summary that becomes the SEO meta description. One or two sentences.
last_updated: 2026-11-18
draft: false
listings:
  - name: Example Co
    url: https://example.com
    affiliate_url: https://example.com/?ref=directory
    summary: What this listing is and why it is useful.
    category: Optional label
  - name: Another Co
    url: https://example.org
    summary: No affiliate_url here, so the link falls back to url.
---

A short introduction that matches what people are searching for.
```

3. Run `make build`. The page is written to `dist/<slug>/index.html` and added to the homepage, `sitemap.xml`, and `robots.txt`.

Front matter rules:

| Field | Required | Notes |
| --- | --- | --- |
| `title` | yes | Used as the page heading and in the document title |
| `description` | yes | SEO meta description, and the blurb on the homepage |
| `last_updated` | yes | `YYYY-MM-DD`. Shown on the page and used as sitemap `lastmod` |
| `slug` | no | Defaults to the file name. Lowercase letters, numbers, and hyphens only |
| `draft` | no | If `true`, the page is skipped by a normal build |
| `listings` | no | List of resources for the directory |

Each listing needs a `name` and either `url` or `affiliate_url`. The outbound link uses `affiliate_url` when it is set, otherwise `url`. Affiliate links get `rel="sponsored noopener noreferrer"`. An optional `summary` and `category` are shown on the card.

`content/pages/sample-directory.md` is a worked example. It is clearly marked as a sample and has `draft: true`, so it is not published by `make build`. Preview it with `make serve-drafts`. Replace or delete it once real pages exist.

Optional: add `content/home.md` if you want Markdown intro copy on the homepage. If that file is missing, the homepage uses `site_description` from `config.yml`.

## Site settings

Edit `config.yml` to set the site name, tagline, description, owner, language, and the affiliate disclosure shown in the footer of every page.

Set these before you publish:

- `base_url` — origin with no trailing slash, for example `https://example.github.io`. Used for canonical URLs and sitemap `<loc>` values.
- `base_path` — leave empty if the site is at the domain root. If it will live in a subdirectory (a typical GitHub Pages project site), set this to that prefix, for example `/everyday-directory`.

## Data files

These are for bookkeeping. They are not part of the built site.

- `data/earnings.csv` — header `date,partner,program,clicks,signups,revenue_usd,notes`. No rows yet; do not invent figures.
- `data/outreach.csv` — header `date,partner,contact,status,terms,notes`.

## Publishing

Do not sign up for a host or deploy from this README. When you are ready to put the site online, build locally with `make build` after setting `base_url` (and `base_path` if needed), then publish the contents of `dist/`. Three common free options:

### GitHub Pages

GitHub Pages will not serve a `/dist` folder from a branch on its own (branch sites only use the repository root or `/docs`). Use a GitHub Actions workflow, or build locally and publish the built files.

**Option A — GitHub Actions (recommended)**

1. Put this project in a GitHub repository and push the default branch (usually `main`).
2. Set `base_url` in `config.yml` to `https://<user>.github.io/<repo>` for a project site, or `https://<user>.github.io` for a user site named `<user>.github.io`. For a project site, also set `base_path` to `/<repo>`.
3. In the repository, go to **Settings → Pages** and set **Source** to **GitHub Actions**.
4. Add a workflow such as `.github/workflows/pages.yml` that checks out the repo, sets up Python 3.11+, installs `requirements.txt`, runs `python build.py`, then uses `actions/configure-pages`, `actions/upload-pages-artifact` (with `path: dist`), and `actions/deploy-pages`. The deploy job needs `pages: write` and `id-token: write` permissions and should use the `github-pages` environment.
5. Push to the default branch (or run the workflow manually). The site will appear at the `base_url` you set.

**Option B — publish a prebuilt folder**

1. Set `base_url` and `base_path` as above, then run `make build`.
2. Copy the contents of `dist/` into a `docs/` folder, or into a `gh-pages` branch at the branch root.
3. In **Settings → Pages**, set **Source** to **Deploy from a branch**, pick that branch, and choose `/docs` or `/` (root) to match where you put the files.

### Cloudflare Pages

1. Set `base_url` in `config.yml` to the hostname you will use, for example `https://<project>.pages.dev` or your custom domain. Leave `base_path` empty unless you are serving from a subdirectory.
2. Two ways to publish:
   - **Direct upload:** run `make build` locally, then in the Cloudflare dashboard go to **Workers & Pages → Create → Pages** and upload the `dist/` folder (no build step; the build directory is the uploaded files).
   - **Git:** connect the GitHub or GitLab repository. Use build command `pip install -r requirements.txt && python build.py`, build output directory `dist`, and production branch `main`. Add an environment variable `PYTHON_VERSION` set to `3.13` (or another 3.11+ version the build image supports).
3. After the first deploy, Cloudflare assigns a `*.pages.dev` subdomain. Update `base_url` to match that hostname or a custom domain, then rebuild so canonical URLs and the sitemap are correct.

### Netlify

1. Set `base_url` in `config.yml` to `https://<site>.netlify.app` or your custom domain. Leave `base_path` empty unless you are serving from a subdirectory.
2. Two ways to publish:
   - **Manual deploy:** run `make build` locally, then in the Netlify dashboard use **Add new site → Deploy manually** and upload the `dist/` folder.
   - **Git:** use **Add new site → Import an existing project**, connect the repository, and set the build command to `pip install -r requirements.txt && python build.py`, the publish directory to `dist`, and `PYTHON_VERSION` to `3.13` (or another 3.11+ version). You can put the same settings in a `netlify.toml` later:

     ```toml
     [build]
       command = "pip install -r requirements.txt && python build.py"
       publish = "dist"

     [build.environment]
       PYTHON_VERSION = "3.13"
     ```
3. After the first deploy, update `base_url` to the real hostname and rebuild so canonical URLs and the sitemap are correct.

## Layout

```
directory-site/
  Makefile              # build, serve, serve-drafts, clean
  build.py              # static site generator
  serve.py              # local preview server
  config.yml            # site name, URLs, affiliate disclosure
  requirements.txt
  README.md
  content/
    pages/              # one Markdown/YAML file per directory page
      sample-directory.md
    home.md             # optional homepage intro (not present yet)
  static/
    css/
      style.css
  data/
    earnings.csv
    outreach.csv
  dist/                 # generated; do not edit by hand
```

+++
date = '2026-05-14T22:27:22-03:00'
title = 'Thetasigma_labs_setup'
tags = []
+++

## Setup

* register domain name
* create github org
* `brew install hugo`
* Create the site

  * `mkdir thetasigma-site`
  * `cd thetasigma-site`
  * `hugo new site .`
  * `git init`
* Instal VSCode extensions:

  ## For making markdown look more like a document than a text file in VSCode, here are the best options

* **Markdown All in One** (yzhang.markdown-all-in-one) — The most popular all-rounder. Better syntax highlighting, auto-formatting, table of contents, keyboard shortcuts for bold/italic, and improved list handling.

  **Markdown Preview Enhanced** (shd101wyy.markdown-preview-enhanced) — Side-by-side preview that renders like a real document with proper typography, math (KaTeX), diagrams (Mermaid), and export to PDF/HTML. Best for "make it look like a document."

  **Marp for VS Code** (marp-team.marp-vscode) — If you want slide-deck style rendering.

  **For editor appearance (not preview):**

  * **Markdown Editor** (zaaack.markdown-editor) — Replaces the text editor with a WYSIWYG view, so headings actually look like headings, not `# text`.
  * **Front Matter CMS** (eliostruyf.vscode-front-matter) — Particularly useful for your Hugo site, since it understands the frontmatter in files like `content/lab-notes/_index.md` and gives a CMS-like editing UI.

  My recommendation: install **Markdown All in One** + **Markdown Preview Enhanced** together. If you want the editor itself (not a preview pane) to look document-like, add **Markdown Editor** . Given you're working in a Hugo site, **Front Matter CMS** is worth a look too.

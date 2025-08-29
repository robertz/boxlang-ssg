# BoxLang SSG

A lightweight static site generator written in BoxLang. It converts Markdown (`.md`) and BoxLang markup (`.bxm`) files into a static website, with layouts, views, collections, tags, and pagination.

## Features
- Fast static builds with simple CLI (`build`, `list`, `help`)
- Markdown and BoxLang markup templates with YAML front matter
- Layouts, views, and partials in `_includes/`
- Collections for pages, posts, tags, plus global JSON data from `_data/`
- Permalinks, slugs, custom file extensions, and pagination
- Passthrough for assets and router files via `ssg-config.json`

## Prerequisites
- BoxLang runtime and CLI (`boxlang` on your PATH)
- Required modules (managed via `requirements.txt`): `bx-jsoup`, `bx-markdown@1.0.0`, `bx-yaml`

## Quick Start

1) Clone and enter the repo

```sh
git clone https://github.com/robertz/boxlang-ssg.git
cd boxlang-ssg
```

2) Install required modules

Use the helper script to read `requirements.txt` and install what’s missing:

```sh
boxlang setup.bx
```

Alternatively, install manually:

```sh
install-bx-module bx-jsoup bx-markdown@1.0.0 bx-yaml
```

3) Build the site

```sh
boxlang ssg.bx build
```

Generated files go to `_site/` (configurable). You can serve `_site/` with any static server. A simple router (`router.bxs`) is also copied into `_site/` for BoxLang’s mini server setups.

## Commands
- `boxlang ssg.bx build`: Build the static site into the configured output folder
- `boxlang ssg.bx list`: List the discovered renderable documents
- `boxlang ssg.bx help`: Show available commands

## Project Structure
- `ssg.bx`: CLI entry and build pipeline
- `setup.bx`: Installs modules listed in `requirements.txt`
- `ssg-config.json`: Build configuration (output dir, passthrough, ignore)
- `_includes/`: Layouts, views, and partials
  - `_includes/layouts/main.bxm`: Base HTML layout
  - `_includes/page.bxm`, `_includes/post.bxm`, `_includes/sidebar.bxm`
- `_data/`: Optional global JSON data (loaded into `collections.global`)
- `posts/`: Example content (Markdown with front matter)
- `assets/`: Static assets (passthrough)
- `router.bxs`: Simple router included in output for dev serving
- `index.bxm`: Example index page showing posts list
- `404.md`: Example 404 page (layout: none)

## Authoring Content

BoxLang SSG reads both `.md` and `.bxm` files. Each file may include YAML front matter to control metadata and output behavior.

Front matter markers:
- Markdown: start/end with `---`
- BoxLang markup: start with `<!---` and end with `--->`

Common front matter keys:
- `title`: Page title
- `description`: Short description
- `type`: Content type (e.g., `page`, `post`) — used for view fallback and collections
- `layout`: Layout file in `_includes/layouts` without extension (defaults to `main`)
- `view`: View/partial in `_includes/` without extension; falls back to `type`
- `slug`: Override file slug (used for posts)
- `tags`: Array of tags; builds `collections.tags` and `collections.byTag`
- `date`: Date the content refers to
- `permalink`: Override output path, e.g., `/tag/{{tag}}.html`
- `fileExt`: Override output extension, e.g., `xml`
- `published`: Boolean to include/exclude from output
- `excludeFromCollections`: Boolean to skip item from collections

Example page (`index.bxm`):

```html
<!---
type: page
layout: main
--->
<bx:output>
<bx:loop array="#collections.post#" item="post">
  <a href="#post.permalink#">#post.title#</a><br />
</bx:loop>
</bx:output>
```

Example post (`posts/example.md`):

```markdown
---
layout: main
type: post
slug: my-first-post
title: My First Post
description: A short summary
tags:
- misc
published: true
---

# Hello
This is my first post.
```

## Collections and Data
- `collections.all`: All templates (except those excluded or parent pagination templates)
- `collections.post`: All items where `type: post` (sorted by date desc)
- `collections.tags`: Unique list of tags across posts
- `collections.byTag[slug]`: Posts grouped by slugified tag
- `collections.global`: Nested structure built from JSON files in `_data/` (mirrors folder structure)

## Pagination
Use `pagination` in front matter with an `alias` and `data` source. The permalink should include `{{alias}}` to expand per page. See `tags.bxm`:

```html
<!---
layout: main
view: page
permalink: /tag/{{tag}}.html
pagination:
  alias: tag
  data: collections.tags
--->
```

In the view you can access `prc.tag` and iterate `collections.byTag[slugify(prc.tag)]`.

## Configuration (`ssg-config.json`)

```json
{
  "outputDir": "_site",
  "passthru": ["router.bxs", "assets"],
  "ignore": []
}
```

- `outputDir`: Build output directory (auto-cleaned each build)
- `passthru`: Files/directories copied verbatim into `outputDir`
- `ignore`: Additional paths to skip when discovering documents

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you’d like to change.

## License
MIT

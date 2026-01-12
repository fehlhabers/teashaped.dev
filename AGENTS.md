# AGENTS.md

This file provides guidelines for AI agents working in this Hugo static site repository.

## Build & Serve Commands

```bash
# Local development (hot reload)
hugo serve
# Or via task runner:
task serve
# Or via mise:
mise run hugo:serve

# Production build
hugo --gc --minify

# Specific build for GitHub Pages
hugo --gc --minify --baseURL "https://teashaped.dev"
```

## Dependencies & Tools

- **Hugo**: v0.123.8+ (uses extended version)
- **Go**: v1.21 (required for Hugo modules)
- **Hextra theme**: github.com/imfing/hextra (managed as Hugo module)
- **mise**: Tool version management (via mise.toml)
- **task**: Task runner (via Taskfile.yaml)

## Content Structure

### File Organization
- `content/` - All markdown content
- `content/blog/{topic}/post.md` - Blog posts (each topic is a directory with post.md)
- `content/_index.md` - Section index pages
- `layouts/partials/custom/` - Custom template overrides
- `static/` - Static assets (images, favicons, etc.)

### Frontmatter Format
All markdown files must include YAML frontmatter:

```yaml
---
title: "Page Title"
date: YYYY-MM-DD  # Required for blog posts
authors:          # Optional, array format
  - name: Kaj Fehlhaber
tags:             # Optional, array for blog posts
  - tag1
  - tag2
type: about       # Optional, for special page types
---
```

### Blog Post Pattern
Create blog posts in subdirectories:
```
content/blog/my-topic/
└── post.md  # Frontmatter includes title, date, authors, tags
```

## Code Style Guidelines

### Markdown & Content
- **Line length**: Keep lines reasonably long (90-120 chars) for readability
- **Emojis**: Use emoji prefixes for headings and emphasis (see services.md for examples)
- **Headings**: Use H2 (##) and H3 (###) for content structure
- **Frontmatter quotes**: Title and description fields use quotes when containing special characters
- **HTML allowed**: Raw HTML is enabled (goldmark.renderer.unsafe: true)

### Naming Conventions
- **Directories**: kebab-case (e.g., `remote-team-playbook`, `gitops-journey`)
- **Files**: lowercase with hyphens, `.md` extension
- **Image references**: `/images/filename.ext` (from static/images/)
- **Blog post file**: Always name the content file `post.md` within its topic directory

### Hugo-Specific Patterns
- **Shortcodes**: Use Hextra theme shortcodes:
  ```hugo
  {{< tabs items="Tab1,Tab2,Tab3" >}}
  {{< tab >}}Content 1{{< /tab >}}
  {{< tab >}}Content 2{{< /tab >}}
  {{< /tabs >}}
  ```

- **Internal links**: Use page references or relative URLs
  ```markdown
  [Link text](/blog/remote-team-playbook/post)
  ```

- **Images**: Place in `static/images/`, reference without `/static/` prefix:
  ```markdown
  ![Alt text](/images/kaj.jpg)
  ```

### Configuration Files
- **Main config**: `hugo.yaml` (not config.toml or config.yml)
- **Module management**: Uses Go modules in `go.mod`
- **Dependabot**: Configured for daily Go module updates in `.github/dependabot.yml`

## Customizations

### Layouts
Minimal custom overrides:
- `layouts/partials/custom/head-end.html` - Analytics scripts, fediverse links, custom meta tags

### Static Assets
- Place new images in `static/images/`
- Favicons in `static/` root (favicon.ico, favicon.svg, etc.)
- Site logo: `static/teashaped-*.png`

## Development Workflow

1. Start dev server: `hugo serve` (runs on http://localhost:1313)
2. Edit content in `content/` directory
3. Changes auto-reload in browser
4. Verify build with `hugo --gc --minify` before committing
5. GitHub Actions auto-deploys from `main` branch to GitHub Pages

## No Testing/Linting

This repository contains no automated tests or linters - it's a static Hugo site. Verify changes by:
- Previewing locally with `hugo serve`
- Building with `hugo --gc --minify`
- Checking generated output in `public/` directory

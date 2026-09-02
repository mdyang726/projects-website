# CLAUDE.md — my-portfolio

Personal aerospace-engineering portfolio. Static Astro site, deployed on Vercel
from `main` (repo: mdyang726/projects-website).

## Stack
- Astro 6.x, no integrations. `astro.config.mjs` is empty by design.
- Node >= 22.12. Package manager: npm.
- Output: static `dist/`. No SSR, no adapter, no runtime.
- No backend, database, API, content collections, or client JS framework.

## Commands
- `npm run dev`     — local dev server (localhost:4321)
- `npm run build`   — production build to dist/
- `npm run preview` — serve the build locally
- `npx astro check` — type/diagnostics check (not currently wired; optional)

## Structure
- `src/pages/index.astro`            — home: hero, project cards, about
- `src/pages/projects/*.astro`       — one self-contained file per project
- `public/images/<project-slug>/`    — page assets, referenced as `/images/...`
- `context/*.md`                     — source notes for page copy; NOT built
- `src/layouts/`, `src/components/`  — UNUSED Astro starter scaffolding

Each page currently owns its full `<html>`/`<head>`/`<style>`. There is no
shared layout. Do not introduce one without it being its own dedicated task
(it is a cross-cutting change — see Boundaries).

## Design system (convention, not enforced)
Colors:  bg #fafaf9 · ink #1a1a18 · body #5f5e5a · muted #b4b2a9 ·
         rule #e0dfd8 · hover #f1efea · badge #FBEFD8 / #92600B
Layout:  #wrapper max-width 860px, 80px side padding (24px below 640px)
Fonts:   DM Serif Display (headings), DM Mono (labels/tags), DM Sans (body)
Blocks:  nav · hero · section-label · content-block · gallery · image-figure
Keep new pages consistent by copying an existing project page.

## Conventions
- Match the surrounding file's indentation and quote style (it varies per file).
- Images: put in public/images/<slug>/, reference absolute (/images/...),
  add loading="lazy", always set alt text (data figures get descriptive alt).
- External links: target="_blank" rel="noopener".
- Copy comes from context/*.md and from the user; never invent project facts.
- Text files are LF (.gitattributes enforces it) — do not reintroduce CRLF.

## Known issues / cleanup backlog
- public/images/node_modules/ and public/images/.astro/ are stray local dirs
  (git-ignored, not tracked) — safe to delete from disk.
- README.md is still the Astro starter template.
- index.astro + de-laval-nozzle.astro footers link to bare https://github.com.
- src/layouts + src/components are dead starter files.

## Boundaries for parallel edits
Safe to edit fully independently (one owner each):
  - src/pages/index.astro
  - src/pages/projects/tvc-rocket.astro       (see improvement queue in agent memory)
  - src/pages/projects/de-laval-nozzle.astro
  - src/pages/projects/projects-site.astro
Additive-only, low conflict:
  - public/images/**
Serialize (single owner, do alone, first or last):
  - any new src/layouts/Layout.astro or shared design-tokens file
  - .gitignore, .gitattributes, package.json, astro.config.mjs, this file

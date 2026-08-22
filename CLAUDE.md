# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this repository is

A static storefront and asset host for **AI Field Guides** — five paid digital
guides sold through Gumroad by Global Vision Enterprises. There is no build
system, no package manager, no tests, and no CI. The repo contains exactly two
hand-written HTML pages plus a large library of media assets (~400 MB) that the
pages and social channels reference.

The two pages:

- `index.html` — the main storefront. Masthead, a 5-card product grid, a
  bundle CTA, a bottom email-capture form, and one `<dialog>` modal per guide
  with a slide-carousel preview and a Gumroad buy link.
- `all-guides-showcase.html` — the "everything included" bundle page linked
  from the storefront's bundle CTA.

## The five guides

Everything in the repo is organized around these five products. Numbered
assets (`booklet2-infographic.jpg`, `card3-cover-1.jpg`, …) use the number;
directory-based assets use the slug.

| # | Slug | Title | Modal id |
|---|------|-------|----------|
| 1 | `9-ai-business-paths` | 9 AI Business Paths | `m-9paths` |
| 2 | `small-business-ai-stack` | The Small Business AI Stack | `m-stack` |
| 3 | `ai-assisted-money-audit` | The AI-Assisted Money Audit | `m-money` |
| 4 | `5-layer-ai-productivity-system` | The 5-Layer AI Productivity System | `m-productivity` |
| 5 | `ai-ecommerce-build-order` | The AI E-Commerce Build Order | `m-ecom` |

Each guide sells for $29; the bundle is $119 (struck from $145). Each guide's
deliverables are: full PDF, companion video, Audio Overview, slide-deck cheat
sheet, and infographic ("25 real assets" across the bundle).

## Directory layout

```
index.html                  Storefront (self-contained: inline CSS + JS)
all-guides-showcase.html    Bundle detail page (same conventions)
images/
  card-previews/            cardN-cover-*.jpg — card-face preview images
  covers/                   Book-cover renders + thumbnails (v2/ = current redesign)
  slides-01-9-ai-business-paths/  Cheat-sheet slides for guide 1 (slide-NN.jpg, zero-padded)
  bookletN-infographic.jpg  Per-guide infographic previews
preview/<slug>/             Full deliverable previews per guide:
  video.mp4, audio.m4a, audio-cover.jpg, book-cover.png,
  infographic.jpg, slides/slide-NN.jpg
avatar/                     AI-avatar marketing videos (blue-hoodie-*.mp4) plus
                            their inputs: reference photos and narration-*.mp3
social-assets/              Daily social-post images: info-<slug>.jpg rotation
                            images, morning-news-YYYY-MM-DD.jpg,
                            evening-offer-YYYY-MM-DD.jpg, article-YYYY-MM-DD.jpg
videos/                     Card teaser videos (e.g. 9aipaths-teaser.mp4)
```

## HTML conventions (both pages)

- **Self-contained single files.** All CSS in one `<style>` block, all JS in
  one `<script>` block at the bottom. No external libraries, fonts, or
  stylesheets. Files intentionally start with `<meta charset>` (no doctype /
  `<html>` / `<body>` wrapper tags) — keep that style.
- **Theme tokens.** Colors are CSS custom properties on `:root`. Light palette
  is the default; the dark palette is defined three times identically: under
  `@media (prefers-color-scheme: dark)`, under `:root[data-theme="dark"]`, and
  light re-asserted under `:root[data-theme="light"]`. When changing a color,
  update every block in **both** HTML files — the two pages share the same
  palette and must stay in sync (paper `#F5F1E8`, ink `#1E2A32`, blue accent
  `#1B4B73`, orange accent `#C1560A`, plus dark equivalents).
- **Typography.** Serif (Georgia) headings, system sans body, `ui-monospace`
  for prices/eyebrows/counters.
- **Modals** are native `<dialog class="pd">` elements with outside-click-to-
  close wiring. Cards open them via `onclick`/`onkeydown` handlers and carry
  `role="button"`, `tabindex="0"`, `aria-haspopup="dialog"` — preserve the
  keyboard/ARIA affordances on anything new.
- **Carousels** are declarative: a `.slidebar` div with either
  `data-prefix`/`data-count`/`data-pad`/`data-ext` (numbered slide sequence)
  or `data-images` (comma-separated list). `data-soon="true"` overlays a
  "Full slide deck preview coming soon" badge — remove it when a guide's real
  slide deck goes live (guide 1 already has its full deck wired; guides 2–5
  still show the two-image placeholder). Card-face `.card-preview` divs
  auto-rotate their `data-images` on a 2.8 s interval.

## External integrations

- **Gumroad** — all purchases. Product links live on the `Buy Now` anchors:
  `housinghub.gumroad.com/l/{9aibusinesspaths, mlbwtk, lmiovc, glunk, lqvwq}`
  and `igkrqe` for the bundle.
- **Lead capture** — every buy click is intercepted by `interceptBuy()`,
  which opens the `m-checkout-capture` dialog, POSTs
  `{name, email, source}` to the Cloudflare Worker at
  `LEADS_ENDPOINT` (`https://gve-leads.globalhousingco.workers.dev`), then
  opens the Gumroad URL. Capture failures must never block checkout (fetch is
  wrapped in try/catch — keep it that way). The `source` strings
  (`buy-9paths`, `buy-bundle`, `storefront-bottom-cta`, …) are analytics
  labels; keep them stable. The Worker's code is **not** in this repo (the
  comment referencing `DEPLOY-STEPS.md` points at that separate project).
- **YouTube** — footer links to the @GlobalVisionEnterprises-f6h channel.

## Media asset conventions

- Slides are `slide-NN.jpg` with zero-padded two-digit numbers, starting
  at 01. Slide counts differ per guide (11–14) — check the directory before
  hardcoding a `data-count`.
- Audio previews are trimmed to ~3:30 with a fade-out — do not commit
  full-length (20+ min) audio.
- Avatar videos follow `blue-hoodie-<topic>.mp4` with matching
  `narration-<topic>.mp3` inputs and reference photos alongside them in
  `avatar/`.
- Dated social images use `YYYY-MM-DD` suffixes and carry the @handle
  watermark.
- Large binaries are committed directly (no Git LFS). Keep individual files
  reasonably sized (current max ~12 MB) and don't re-encode existing assets
  without reason — every replacement bloats git history.

## Development workflow

- **Preview:** just open the HTML files in a browser, or
  `python3 -m http.server` from the repo root. No build step exists.
- **Verify by hand:** there are no tests. After editing a page, check it in
  both light and dark themes, open each affected modal, and click through the
  carousels. Anything referencing an image/video path should be checked
  against the actual file on disk (paths are relative to repo root).
- **Branches:** `main` is the deployed state. Do feature work on a branch and
  push with `git push -u origin <branch>`; never force-push `main`.
- **Commit messages:** descriptive one-line imperative summaries of the
  user-visible change, in the style of the existing history (e.g. "Add
  @handle watermark to morning news + evening offer posts").

## Things to watch out for

- The two HTML files duplicate the palette, the `interceptBuy` /
  checkout-capture machinery, and the dialog close wiring. A change to any of
  those almost always needs to land in both files.
- Guide 2's modal says "30-day money-back guarantee" like the others, but
  guide 1 says "7-day" — treat guarantee copy as owner-decided; don't
  "normalize" it without being asked.
- Prices appear in multiple places (cards, modals, bundle block on both
  pages). A price change touches all of them.

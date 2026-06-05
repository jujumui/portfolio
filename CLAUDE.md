# Portfolio Site — Architecture & Design Decisions

Single-page static site. Plain HTML with inline CSS + JS. No build step, no framework.
Designed to be served directly from the repo root via GitHub Pages.

## File Structure

```
index.html          — the entire site (HTML + CSS + JS, all inline)
.nojekyll           — tells GitHub Pages to skip Jekyll processing
CNAME               — custom domain for GitHub Pages
images/             — place Art_Hero_Big.png here
fonts/              — place licensed .woff2 files here (see fonts/README)
```

## GitHub Pages Setup (manual, one-time)

Go to repo Settings → Pages → Deploy from a branch → select `main` / root.
DNS records at your registrar: apex A records to GitHub's IPs + a `www` CNAME
pointing to `<username>.github.io`. Update CNAME to your real domain before enabling.

## Design Decisions to Preserve

### Layout Rail

`.container` is `max-width: 1136px`, centered, with **no** horizontal padding on desktop.
Tablet/mobile (`max-width: 991px`) gets `24px` left/right padding.
`.div-nav` additionally gets `24px` left/right padding on **desktop only** (`min-width: 992px`)
so nav items don't hit the screen edge at narrower desktop widths (992–1136px).

### Hero — Desktop

Nav + hero together fill exactly the viewport:
- Header height is controlled by `--nav-h: 89px`
- Hero uses `min-height: calc(100vh - var(--nav-h))` so it fills the rest

`.text-hero` is `position: absolute; left: 64px; top: 50%` with
`transform: translateY(calc(-50% - 76px))` — vertically centered and lifted 76px.
Width is fixed at `540px` so it never re-wraps as the viewport narrows.

`.art-hero` (the illustration) is `614px` wide with `translateY(-20px)` for an optical lift.
It is `justify-content: flex-end` so it sits on the right.

### Hero — Tablet/Mobile (max-width 991px)

Switches to stacked flow:
- Image on top (`order: 1`), headline below it (`order: 2`)
- `.text-hero` becomes `position: static; transform: none; width: 100%; text-align: center`
- Hero `min-height` resets to `0` — it is **not** forced to `100vh` on small screens

This is the correct responsive behavior. Do not revert it.

### Mobile (max-width 479px)

- Nav links hidden; hamburger button shown (`display: inline-flex`)
- Opening the menu adds `.open` to `#navLinks`, rendering it as an absolute dropdown
- Headline shrinks to `34px`

### Sticky Nav + Scroll Behavior

`.div-nav-section` is `position: sticky; top: 0`.

On scroll **down**: gains `.nav-hidden` → `translateY(-100%)` + `opacity: 0` (fades up and out).
On scroll **up**: `.nav-hidden` removed → slides back in.

Background is `transparent` at the very top; gains `.nav-solid` (sets `background: var(--bg-start)`)
once `scrollY > 10`. This avoids an abrupt background flash on page load.

### Page Color Transition (next section)

There is a `.next-section` placeholder (min-height 1000px) below the hero.

A single `position: fixed; inset: 0` layer `.bg-overlay` (background `#D9BDA3`, `z-index: -1`)
covers the whole viewport. An `IntersectionObserver` on `.next-section` with
`rootMargin: "0px 0px -40% 0px"` fades the overlay **in** when the section is 40% into
the viewport and **out** when it leaves.

**Why this approach:** a per-section background caused a visible seam at the boundary.
A scroll-linked gradient felt mechanical. A full-page overlay that fades once per crossing
reads as a deliberate, editorial color change. Do not replace this with a CSS gradient or
per-section background.

### Fonts

Self-hosted via `@font-face` from `fonts/`. Three files required (see `fonts/README`).
Font stacks include `"Test Martina Plantijn"` / `"Test Geograph"` as first choices — these
are the Figma trial font names. They will resolve if the trial fonts happen to be installed
locally, but they are **not licensed for production**. Purchase webfont licenses and drop the
files in `fonts/` before going live.

Graceful fallbacks: Georgia / system sans — the site looks correct without the custom fonts.

### Image

Referenced at the relative path `images/Art_Hero_Big.png`. Do not change this path.
A 404 on the image is expected until the file is added; do not "fix" it by removing the
`<img>` tag or changing the path.

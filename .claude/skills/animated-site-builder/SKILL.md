---
name: animated-site-builder
description: "Build and ship a modern, heavily-animated single-page site (restaurant, portfolio, landing page, brand, product) with Astro and deploy it to GitHub Pages. Battle-tested recipe: all CSS/JS inlined for GitHub Pages subpath compatibility, a catalogue of CSS keyframe + SVG animations, JS-driven page transitions, realistic layered SVG illustrations, self-hosted images, Playwright visual regression, and a gh-pages worktree deploy flow. Use when the user asks to create/build a website, landing page, one-pager, animated site, restaurant/portfolio/brand site, or to deploy an Astro site to GitHub Pages. Includes the exact fixes for the black-screen, CSS-not-loading, and push-rejected failures learned the hard way."
---

# Animated Site Builder

A reusable, battle-tested recipe for building a modern, richly-animated single-page
site with **Astro** and shipping it to **GitHub Pages**. Distilled from building the
HIO Sushi Lab site end-to-end. Follow it and you skip re-discovering every trap.

## When to Apply

- "Build me a site for X" (restaurant, portfolio, brand, product, landing page)
- "Make it modern, with animations"
- "Deploy it so I can see it on mobile" / "put it on GitHub Pages"
- Any Astro static site that must work under a `/repo-name/` subpath

Pair this with **ui-ux-pro-max** for palette/typography/style choices. This skill
covers the *build system, animation engineering, and deployment* — the parts that
break silently. ui-ux-pro-max covers *what it should look like*.

## The One Rule That Matters Most

**On GitHub Pages under a subpath (`user.github.io/repo/`), inline EVERYTHING.**

Astro extracts CSS to `/_astro/*.css` and JS to `/_astro/*.js` with absolute paths.
Under a project subpath those 404, so the page renders as unstyled HTML (the classic
"it works locally, plain text on mobile" bug). The fix:

- All CSS goes in `<style is:inline>` inside `<head>`
- All JS goes in `<script is:inline>`
- All asset URLs are **relative**: `src="img/hero.jpg"`, `href="favicon.svg"` — never
  leading-slash absolute (`/img/hero.jpg` breaks under the subpath)
- Build one self-contained `src/pages/index.astro`. One file, no imports for the page.

This single decision prevents ~80% of the pain. See `references/pitfalls.md` for the
full list of failure modes and their exact fixes.

## Build Workflow

1. **Scaffold** — an Astro project already exists (this repo is one). Work in
   `src/pages/index.astro`. Keep the `<style is:inline>` and `<script is:inline>`
   discipline from line one.
2. **Structure** — hero → philosophy/about → signature/features → showcase →
   menu/catalogue → reviews/social proof → reservation/CTA → footer. Each `<section>`
   is `position:relative; overflow:hidden` so absolutely-positioned decorations clip.
3. **Animate** — layer the animation catalogue (`references/animation-toolkit.md`):
   scroll-reveal via IntersectionObserver, CSS keyframe loops, SVG path-draw, floating
   decorations, and a JS-driven page transition.
4. **Illustrate** — for food/product art, prefer **real self-hosted photos**; when
   drawing SVG, use layered radial gradients not flat fills (see toolkit "Realistic SVG").
5. **Verify** — screenshot with Playwright at every visual change (see below). Never
   trust "it built" — a missing `</style>` builds fine and blanks the page.
6. **Deploy** — push to the feature branch (CI deploys) AND/OR use the fast
   gh-pages worktree flow. See `references/deploy-github-pages.md`.

## Verify Before You Ship (non-negotiable)

A successful `npm run build` does NOT mean the page renders. A dropped `</style>`
turns the whole body into CSS text and the build still passes. Always screenshot:

```js
// /tmp/shot.js
const { chromium } = require('playwright');
(async () => {
  const b = await chromium.launch();
  const p = await b.newPage({ viewport:{width:1280,height:900} });
  await p.goto('file://' + process.cwd() + '/dist/index.html');
  await p.waitForTimeout(3500);
  // Sanity check the page actually rendered (catches the blank-screen bug):
  const kids = await p.evaluate(() => document.body.children.length);
  console.log('body children:', kids); // 0 == broken (HTML parsed as CSS)
  await p.screenshot({ path:'/tmp/shot.png' });
  await b.close();
})();
```

Chromium is pre-installed at `/opt/pw-browsers/chromium`; do NOT run
`playwright install`. Read `/tmp/shot.png` back to actually look at it.

## Real Data & Assets

- **Find real images/menu**: for a WordPress site, hit `https://SITE/wp-json/wp/v2/media?per_page=100`
  to enumerate every uploaded image URL, then download the ones you want.
- **Self-host, don't hotlink**: save images into `public/img/` and reference them
  `src="img/name.jpg"`. Hotlinked external images fail behind the agent proxy
  (`ERR_CERT_AUTHORITY_INVALID`) and are unreliable on the live site.
- Convert `.webp` → `.jpg` with Python PIL; crop to focus a subject
  (`Image.open(f).crop((l,t,r,b)).save(out, quality=85)`).

## Reference Files

- `references/deploy-github-pages.md` — CI workflow + the fast gh-pages worktree deploy,
  and how to handle push rejections.
- `references/animation-toolkit.md` — copy-paste catalogue: scroll reveal, keyframe
  library, SVG path-draw, floating decorations, JS-driven page transition, realistic SVG.
- `references/pitfalls.md` — every failure we hit and the exact fix.

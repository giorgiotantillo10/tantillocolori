# Pitfalls & Exact Fixes

Every failure we actually hit building HIO Sushi Lab, and the fix. Read this before
debugging — the symptom is usually one of these.

## Blank / black screen (page renders nothing)

Two distinct causes, same symptom:

1. **Missing `</style>` tag.** The browser then parses all following HTML as CSS text,
   so `document.body.children.length === 0`. `npm run build` still SUCCEEDS. Diagnose
   with the Playwright body-children check (see SKILL.md "Verify"). Fix: restore the
   `</style>` before `</head>`. Guard against it: when editing near the style block,
   never let a multi-line Edit swallow the closing tag.
2. **CSS `animation ... forwards` chaining in a page transition.** A phase that ends
   with a filled state (`forwards`) blocks the next phase from running, freezing on a
   dark panel. Fix: drive multi-phase transitions with `setTimeout` + class toggles and
   CSS `transition` (not `animation`). See animation-toolkit §5.

## Styling works locally, plain unstyled HTML on GitHub Pages / mobile

Astro extracted CSS/JS to `/_astro/*` with absolute paths that 404 under the
`/repo-name/` subpath. Fix: `<style is:inline>` + `<script is:inline>`, and make every
asset URL relative (`favicon.svg`, `img/x.jpg` — never `/favicon.svg`, `/img/x.jpg`).

## External images don't load (ERR_CERT_AUTHORITY_INVALID)

Hotlinked third-party images fail behind the agent proxy and are unreliable live. Fix:
download into `public/img/` and reference relatively. Find a WordPress site's images via
`https://SITE/wp-json/wp/v2/media?per_page=100`.

## GitHub Pages Action fails on configure-pages

`actions/configure-pages@v5` needs Pages manually enabled first. Fix: use
`peaceiris/actions-gh-pages@v4` (pushes to `gh-pages` branch) and set repo Pages source
to that branch once. See deploy-github-pages.md.

## Jekyll eats your `_astro`/underscore assets on Pages

Add `touch dist/.nojekyll` in the build (and in the worktree deploy) so Pages serves
files/folders beginning with `_`.

## git push rejected — "Updates were rejected (fetch first)"

Someone/CI pushed to that ref. For **gh-pages** (a build artifact) `git push --force` is
fine, or `git pull --rebase origin gh-pages` then push. For the **source feature branch**
prefer `git pull --rebase` then push — never force-destroy source history.

## `git worktree remove` prints a getcwd error / "cwd was reset"

Harmless. The shell's working dir was inside the removed worktree. It does not affect the
push. Just `cd` back to the repo root.

## The page "built fine" so I shipped it — and it was broken

`npm run build` passing proves nothing about rendering. A dropped tag, a bad relative
path, or a JS error all build clean. **Always** screenshot with Playwright and actually
read the image back before deploying. This is the single most valuable habit.

## Page-transition "grab" (or any timed motion) never visibly happens

The moving element's transition duration outran the phase schedule — the next phase
started before it finished traveling. Fix: make each phase's `setTimeout` ≥ the
transition duration of the motion it depends on. Verify by screenshotting mid-transition
at several offsets (`page.click(...)` then timed `screenshot`s).

## Section label and title overlap on one line

An inline/`inline-flex` label (`.lbl`) sits on the same line as an `inline-block` title.
Fix: make the label `display:flex` (block-level) so the title wraps below; add a centered
variant for centered sections.

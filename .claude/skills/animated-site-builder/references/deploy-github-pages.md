# Deploying an Astro site to GitHub Pages

Two paths. The CI workflow is the durable one; the worktree flow is for fast iteration.

## 1. CI workflow (set up once)

`.github/workflows/deploy-pages.yml`:

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [ <your-feature-branch>, main ]
  workflow_dispatch:
permissions:
  contents: write
concurrency:
  group: pages
  cancel-in-progress: true
jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - run: npm ci
      - run: npm run build
      - run: |
          rm -rf dist/_worker.js dist/_routes.json
          touch dist/.nojekyll
      - uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
          force_orphan: true
```

Why `peaceiris/actions-gh-pages@v4` and NOT `actions/configure-pages@v5`: the official
Pages action requires Pages to be manually enabled in repo settings first, which fails
in an automated session. `peaceiris` just pushes to the `gh-pages` branch — set the
repo's Pages source to "Deploy from branch → gh-pages" once and it works.

Key steps that are easy to forget:
- `rm -rf dist/_worker.js dist/_routes.json` — these are Cloudflare adapter artifacts
  that confuse Pages.
- `touch dist/.nojekyll` — without it, Jekyll on Pages ignores files/folders starting
  with `_`, breaking assets.

Live URL: `https://<user>.github.io/<repo>/`. Pages takes 1-2 min to refresh after a push.

## 2. Fast gh-pages worktree deploy (for immediate updates)

Deploy the freshly-built `dist/` without waiting for CI:

```bash
npm run build

# Add a worktree checked out to gh-pages (create it if the remote branch is new)
git worktree add /tmp/ghw gh-pages 2>/dev/null \
  || git worktree add /tmp/ghw -B gh-pages origin/gh-pages

cd /tmp/ghw
cp -r /path/to/repo/dist/. .
rm -rf _worker.js _routes.json && touch .nojekyll
git add -A && git commit -m "Deploy: <what changed>"
git push --force origin gh-pages          # force is fine: gh-pages is a build artifact
cd /path/to/repo
```

Notes:
- After `git worktree` commands the shell prints `Shell cwd was reset` — harmless.
- Reuse one worktree dir across deploys; `cp -r dist/. .` overwrites in place.
- `--force` is safe here because gh-pages only ever holds generated output.
- If a plain `git push` is rejected as non-fast-forward, either `git pull --rebase
  origin gh-pages` then push, or `git push --force` (gh-pages) — for the SOURCE feature
  branch prefer rebase, never force-blow-away source history.

## 3. Also commit the source

The worktree flow only publishes `dist/`. Always ALSO commit the real change to the
feature branch so the source and the live site stay in sync:

```bash
git add src/pages/index.astro public/
git commit -m "..."
git push -u origin <your-feature-branch>
```

# Hosting this app on Dr. Cole's Math Lab

This folder holds the change that adds the explorer to
[`drcolesmathlab/drcolesmathlab.github.io`](https://github.com/drcolesmathlab/drcolesmathlab.github.io)
as **app 04**, ready to apply.

It is here rather than in that repository because the session that built it was
scoped to `drstockinvesting` and its git proxy would not issue a credential for a
repository under another owner. The change itself is finished and verified — only
the push was blocked.

## What the patch does

| File | |
| --- | --- |
| `tools/vendor-squares-cubes.js` | new — copies this repo's `index.html` into `apps/squares-cubes/`, lifting the inline `<style>` into `styles.css` |
| `tools/site.config.js` | new `apps` entry: slug `squares-cubes`, category `--tc`, no storage keys, no third-party code |
| `squares-cubes.html` | new page, built from `transform-lab.html` with the prose rewritten |
| `tools/og-cards.html` + `assets/og-squares-cubes.png` | new social card |
| `apps/squares-cubes/` | the vendored app |
| the other three pages | regenerated: numbering `03` → `04` and the pager wired to the new page |

### Why the stylesheet gets split out

Upstream is deliberately one double-clickable file. But `apply-site-chrome.js`
injects the `@font-face` blocks by rewriting a payload stylesheet, and it only
looks for `apps/<slug>/styles.css` or `style.css`:

```js
for (const name of ['styles.css', 'style.css']) {
  const abs = path.join(appsDir, slug, name);
  if (!fs.existsSync(abs)) continue;   // <- a single-file app is skipped, silently
```

Vendored whole, this would be the one app that quietly stops following when the
site's fonts change. The split hands its fonts back to the generator. The app
reads no styles back from the DOM, so it is behaviour-neutral.

## Applying it

The patch was built on `682b721` ("Update README deploying section"). From a
checkout of the site repo:

```bash
git checkout -b claude/add-squares-cubes-explorer
git am /path/to/0001-add-squares-cubes-explorer-to-math-lab-site.patch
```

Then re-run the repo's own gates, which is how this was verified:

```bash
node tools/apply-site-chrome.js   # must be idempotent — run twice, no diff
node tools/check-external.js      # must report zero external origins
```

### Keeping the vendored payload current

`apps/squares-cubes/` inside the patch is a copy of this repo's `index.html`, so
it goes stale every time that file changes. Both of its hunks are whole-new-file
additions, which carry no context to invalidate, so they can be regenerated in
place without touching the rest of the patch — the binary card, the five context
diffs against blobs this repo does not hold, and the trailer are spliced through
byte-for-byte, and only the diffstat is recomputed.

The recipe is the vendor script's lift, followed by the one edit
`apply-site-chrome.js` makes: it stamps its own header comment over the upstream
one inside the `FONTS` block, leaving the base64 payload identical. Regenerating
is worth a check that the same recipe still reproduces the hunks already in the
patch from the previous `index.html` — if it stops doing so, the site's
generator has changed and the patch needs re-cutting against the site repo
rather than editing.

If this repo has moved on further than that — anything outside
`apps/squares-cubes/` — skip the patch and just re-run the vendor script, which
reproduces the same directory from source:

```bash
node tools/vendor-squares-cubes.js /path/to/exploration_perfect_squares_and_cubes
node tools/apply-site-chrome.js && node tools/check-external.js
```

## One step left to run

`node tools/build-og-cards.js` could not run in the authoring container: it probes
fixed Chrome paths, and the container runs as root, which modern Chromium refuses
without `--no-sandbox`. `assets/og-squares-cubes.png` in the patch was rendered
separately at the same 1200×630 spec and the other four cards are untouched, so
nothing is missing — but re-running the script on a normal machine is the way to
confirm all five cards agree. The script itself is unmodified.

# Squares & Cubes Explorer

A single-file classroom app that ties **n²/√n** and **n³/∛n** to the shapes they
describe: a side length on the left, the assembled figure on the right, and an
arrow in the middle naming the operation that joins them.

Open `index.html` in any modern desktop browser. No server, no build step, no
dependencies — copy the file onto a shared drive or email it and it just runs.

## Level 1 · Explore (built)

| Part of the screen | What it does |
| --- | --- |
| **Left** | A slider sets the side length (squares 1–20, cubes 1–10) and lays that many units in a line. |
| **Middle** | An arrow pointing **→** for squaring/cubing or **←** for rooting, with the notation underneath: `5²`, `5³`, `√25`, `∛125`. |
| **Right** | The n×n square or n×n×n cube, with a counter for the total: `5 × 5 × 5 = 125`, and for cubes `5 layers × 25 = 125`. |

Details worth pointing out to students:

- **Growing a square adds an L.** Raise the slider by one and the new row +
  column + corner flash — that L-shape is the `2n + 1` between `n²` and `(n+1)²`.
- **Explode layers** (cubes) fans the cube into its `n` cross-sections, each
  labelled `n × n = n²`, so `n³` reads as *n layers of n²*.
- **Drag the cube** to rotate it; **Reset view** returns to the opening angle.
- Flipping the arrow to **←** swaps which side is labelled *Start with* and
  which is *Answer*, so it is always clear which quantity is the given one.

Keyboard: `S` squares, `C` cubes, `E` explode, `R` reset view, `D` flip
direction; arrow keys move the slider once it has focus.

## How the drawing works

- **Square** — a CSS grid of unit `div`s, sized to fill the panel it is measured
  against.
- **Cube, assembled** — CSS 3D transforms, one `div` per unit cube with six
  faces. Only the **shell** is built (any coordinate on the outside): identical
  from every angle, and 488 cubes instead of 1000 at n = 10.
- **Cube, exploded** — the `n` cross-sections, each a flat plate of unit squares.
  The fanned stack is scaled to fit the panel by projecting its bounding box at
  the current viewing angle (`fitScale`).

## Adding the next level

`index.html` is sectioned: `CONFIG` (per-mode numbers and notation), `state` +
`setState` (a diffing update so rotation never rebuilds the cube), the builders
(`buildRow`, `buildSquare`, `buildCubeShell`, `buildCubeLayers`), the renderers,
and a `LEVELS` registry at the bottom.

A new level is an entry in `LEVELS`, selected with `?level=2`:

```js
2: {
  id: 2,
  title: "Level 2 · Build it",
  setup(state) { /* swap in the level's own controls */ },
  validate(state) { /* check the student's answer */ }
}
```

The builders and the notation/counter renderers are meant to be reused as-is —
a level that has students *build* the square, or type the exponent or radical,
should only need to change the controls around them.

Planned: build-the-figure, enter-the-notation, and imperfect roots (where a
total that will not form a full square lands *between* two perfect squares).

## Theme

The app wears the **Dr. Cole's Math Lab** theme, so it matches the other apps in
the collection (Transform Lab, Balancing Act, Tiger Trail). Surface and ink
tokens are carried verbatim from that site's `tools/site.config.js`, where they
are contrast-measured to WCAG 2.1 AA; squares take the cyan accent and cubes the
orange one. It is **dark only** — the Math Lab palette is a dark one, and there
is deliberately no `prefers-color-scheme` branch to flip.

Nunito and Space Mono ship inside `index.html` as base64 `data:` URIs, between
the `FONTS:START` / `FONTS:END` markers. That is not decoration: Chrome fetches
`@font-face` in CORS mode, and from a `file://` page the origin is opaque, so a
relative `.woff2` fails silently and the page renders in Helvetica with nothing
to signal that anything went wrong. A `data:` URI has no origin to check. Both
faces are SIL Open Font License; the source files and licence text live in the
Math Lab repo under `assets/fonts/`.

The one rule inherited from that site holds here too: **no external-origin
requests**. Everything the page needs is in the file.

## Browsers

Built and checked on desktop Chromium at 1440×900. The layout collapses to a
single column below 900px and all pointer handling uses Pointer Events, so it is
usable on a phone — but mobile is a working fallback, not a finished design.

# samardala.com

An interactive valley built around **samardala** (*Allium siculum* subsp.
*bulgaricum*, Bulgarian honey garlic) — the herb of the Stara Zagora region.
Eight fields are marked with carved posts; дядо Жельо walks between them and
tells you one thing at each.

## Stack

There is no build step. `index.html` is a single self-contained page that loads
three.js through an import map and runs directly in the browser.

- **three.js 0.180.0** — vendored in `vendor/`, not loaded from a CDN
- **No npm install, no bundler, no node_modules**
- Deployed as static files

### Files

```
index.html                 the whole thing — markup, CSS, shaders, simulation
og.png                     1200x630 social card (must sit at the site root)
vendor/three.module.min.js
vendor/three.core.min.js   three.module imports this; they travel together
LICENSE-lentils.txt
.gitignore                 hidden on macOS — Cmd+Shift+. to see it
```

## Local development

Import maps and ES modules will not run from `file://`. Serve the folder:

```
python3 -m http.server 8000
```

Then open http://localhost:8000

## Deployment

GitHub -> Vercel, Framework Preset **Other**, no build command, output directory
left empty (the repo root is served as-is). Custom domain via GoDaddy DNS: an
A record on `@` and a CNAME on `www`, values taken from Vercel's domain screen.

`og.png` must be at the root, because the Open Graph tags reference it by
absolute URL (https://samardala.com/og.png).

## Origin

The procedural terrain, wind field and grass simulation are forked from
`claude-opus-5-ghibli` by Lentils, MIT licensed. See `LICENSE-lentils.txt`.

Inherited architecture worth knowing before editing:

- **Wind** (§5) — Ornstein-Uhlenbeck meander of the mean flow, a turbulence
  cascade advected with it, discrete gust cells, a logarithmic boundary layer.
  Baked once per frame into one 256² texture that every other system reads,
  with a CPU mirror (`windAtJS`) so audio and camera agree with the GPU.
- **Grass** (§6) — blades as quadratic Béziers solved for quasi-static
  equilibrium of gravity, wind and Hookean recovery (Jahrmann & Wimmer 2017).
  Four LOD rings, ~804k blades. The instance buffer is shuffled, so any prefix
  is a uniform random sample and thinning is just a lower instance count.
- **Quality** — four presets scaling grass density, shadow map, wind texture
  and tessellation together. `autoQuality()` steps down below 34 fps.

## The samardala

Eight fields in the `SAMARDALA` table: position, radius, and the line дядо
Жельо speaks there. `SAM_HELLO` is his introduction (once, queued ahead of the
first field's line); `SAM_DONE` is the closing speech.

The mask is evaluated analytically in the grass vertex shader (`samardalaAt`)
from `uniform vec4 uSamardala[8]` — centre in `xy`, radius in `z`, strength in
`w`. Not a texture: at 4.7 m per meadow texel a 20 m field would be four texels
across, and the ragged edge is the point. Lobing noise ramps in toward the rim
so the core stays solid.

Inside a field the blade is 1.2x taller, 1.95x wider, three to five times
stiffer, rings at twice the frequency with a fifth of the amplitude, grows no
seed heads, and takes a glaucous blue-green hue path. One stem in twelve is a
flowering scape carrying the drooping bells. The effect is that a gust lays the
hay over and the samardala refuses — you find a field by how it fails to move.

Two constraints are enforced at boot by `clearStands()`, because both have
already caused bugs:

1. **Clear of the railway and the viaduct.** The deck is a conditional ground
   surface; a field inside its 144 x 7.6 m corridor teleports you onto the
   bridge. Measured laterally against the bridge axis, not as a point distance.
2. **Clear of each other.** `updateSamardala` returns the first field you are
   inside, so a field within another is unreachable and plays its neighbour's
   line.

The console prints where every field ends up, flags any that moved, and shouts
if one is still near the line.

## Navigation

- **Desktop** — WASD, mouse look, F to fly, M for the field index, Esc releases
- **Click to travel** — pins in the view or entries in the index; the glide
  arcs over the terrain and the viaduct, since a straight run at head height
  collided with the bridge and stranded you
- **Touch** — drag anywhere to look, tap a letter in the bottom dock to travel.
  No sticks. Starts at Low preset, 55% density, 72% render scale, pinned

## Still to do

- [ ] Replace the Japanese village, viaduct and railway (§9, §9b) with
      Sarnena Sredna Gora furniture
- [ ] Repalette (§0b) for Bulgarian spring light — the sun is still late August
- [ ] An HTML content layer underneath the canvas, for search engines
- [ ] The shop

# Odysseus FX

A single self-contained HTML page with **25 live, interactive visual effects** — particle flow fields, reaction–diffusion, metaballs, boids, fluid-ish shaders, raymarched blobs and more. No build step, no bundler, (almost) no dependencies. Just open the file.

> The 40-line magic show. Every panel is a running effect — not a screenshot. Hover, drag, and poke them.

**Live demo:** `https://<your-username>.github.io/odysseus-fx/` *(after you enable Pages — see below)*

---

## What's inside

Everything lives in one file: [`index.html`](index.html). One `<canvas>` per tile, a shared animation loop, a theme system, and the math.

### Category A — vanilla (18, no dependencies)
Pure Canvas 2D + plain JavaScript. Ranked by trick-to-code ratio.

1. Flow field · 2. Reaction–diffusion (Gray–Scott) · 3. Metaballs · 4. Boids · 5. Falling sand · 6. Julia set · 7. Plasma · 8. Tixy field · 9. Digital rain · 10. Starfield warp · 11. Conway's Life · 12. Water ripple · 13. Fireworks · 14. Spectrum bars (Web Audio, optional mic) · 15. Error-diffusion dither · 16. Harmonograph · 17. Constellation · 18. Sine dot grid

### Category B — GPU & libraries (7)
Raw WebGL shaders, plus two tiles that pull a library from a CDN (marked `lib`).

1. Raymarched blobs *(WebGL)* · 2. Curl-noise particles *(WebGL)* · 3. Infinite tunnel *(WebGL)* · 4. Domain-warped ink *(WebGL)* · 5. Mesh gradient *(WebGL)* · 6. Point-cloud morph *(three.js)* · 7. Stagger grid *(anime.js)*

---

## Features

- **Interactive.** Drag the raymarcher to orbit, paint into the sand and Game of Life, click to drop ripples and launch fireworks, move the cursor to stir the flow field or steer the Julia constant.
- **Theme switcher** in the top bar — Odysseus (default), Void, Abyss, Ember, and a Paper light mode. Every effect re-reads the palette and recolors instantly, so you can see each one on different backgrounds.
- **"View the trick"** — each tile has an expander showing the key idea in code.
- **Built to behave.** Only on-screen tiles animate (IntersectionObserver), there's a global pause, `prefers-reduced-motion` is respected (starts paused with an opt-in), and an FPS readout sits in the footer.
- **Graceful degradation.** The two `lib` tiles fall back to a small note if they can't reach the CDN; WebGL tiles do the same where WebGL is unavailable. CPU-heavy per-pixel effects render at reduced internal resolution and upscale.

---

## Run it locally

It's a static file — just open `index.html` in a browser. Or serve it (nicer for the Web Audio mic permission and CDN tiles):

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploy to GitHub Pages

**With the GitHub CLI (fastest):**

```bash
gh repo create odysseus-fx --public --source=. --push
```

Then turn on Pages: **Settings → Pages → Build and deployment → Source: "Deploy from a branch" → Branch: `main` / `/ (root)` → Save.**

**With plain git** (after creating an empty repo at <https://github.com/new>):

```bash
git remote add origin https://github.com/<your-username>/odysseus-fx.git
git branch -M main
git push -u origin main
```

Your site appears at `https://<your-username>.github.io/odysseus-fx/` within a minute or two. The included `.nojekyll` file tells Pages to serve everything as-is.

---

## How it's organized

```
odysseus-fx/
├── index.html     # the entire gallery — framework + 25 effects
├── .nojekyll      # serve static files as-is on GitHub Pages
├── .gitignore
├── LICENSE
└── README.md
```

Inside `index.html`, effects self-register with a small framework:

```js
FX.register({
  id, name, cat: 'vanilla' | 'gpu', rank, trick, src,
  mode: '2d' | 'gl' | 'raw',
  setup(env) { /* ... */ return { draw(t), teardown() }; }
});
```

`env` hands each effect its canvas/context, size, a live `pal()` palette getter (so theme switches recolor it), mouse state, and a place to mount controls. Adding a 26th tile is one more `FX.register({...})`.

---

## Credits

The techniques are classics from the creative-coding world; this is an original from-scratch implementation of each. Worth a look:

- **Daniel Shiffman / The Coding Train** — flow fields, boids, reaction–diffusion, fractals
- **Inigo Quilez** — raymarching, SDFs, domain warping, noise (iquilezles.org)
- **Karl Sims** — the canonical reaction–diffusion tutorial
- **Craig Reynolds** — *Boids* (1986/87), the original flocking model
- **Martin Kleppe** — [tixy.land](https://tixy.land), the 16×16 one-formula dot field
- **Pavel Dobryakov** — WebGL fluid simulation (inspiration for the shader tiles)
- Libraries: [three.js](https://threejs.org) · [anime.js](https://animejs.com)

Fonts: Space Grotesk + JetBrains Mono (Google Fonts).

## License

MIT — see [LICENSE](LICENSE). The code here is yours to do whatever with; the linked libraries keep their own licenses.

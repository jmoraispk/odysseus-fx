# The Browser Wizardry Catalog: Advanced Visual & Animation Techniques for "Odysseus FX"

## TL;DR
- **The modern vanilla web platform alone now ships the majority of the "1% of websites" effects** — scroll-driven animations (`animation-timeline`), View Transitions, `@property`-typed animatable gradients, SVG `feTurbulence`/`feDisplacementMap` distortion, the gooey `feColorMatrix` filter, Canvas particle systems, Web Audio reactivity, and raw WebGL/WebGPU shaders — with zero external dependencies, which is ideal for a dark, glowing, particle-heavy gallery like Odysseus FX.
- **The biggest "wow per byte" vanilla wins** are: the SVG gooey filter (blobby teal merges), `@property` conic-gradient glow borders, scroll-driven parallax/reveal in pure CSS, and `feTurbulence` displacement for organic distortion — all "easy-to-medium" and instantly on-brand for cyan/coral neon.
- **Libraries earn their weight for the hardest tier**: GSAP + ScrollTrigger (sequencing), Three.js/R3F + `UnrealBloom` (the glow look), Lenis (smooth scroll sync), Rive/Lottie (vector motion), and curtains.js/OGL/Shadertoy patterns (WebGL-on-DOM). Use them when native APIs hit a ceiling, not before.

---

## Key Findings

The web platform has quietly absorbed most of what used to require heavy libraries. Scroll-driven animations, View Transitions, typed custom properties, and SVG filters are now baseline or near-baseline in Chromium/WebKit, and they run **off the main thread** for free — a decisive performance advantage over JavaScript scroll listeners. For a dark, glowing effects gallery, the single highest-leverage decision is to build the easy/medium tier in pure CSS+SVG+Canvas and reserve libraries (GSAP, Three.js, Lenis) for cinematic sequencing and true 3D/WebGL glow.

Two structural notes for Odysseus FX's aesthetic:
- **Glow on dark** comes from three reliable recipes: CSS (`box-shadow`/`drop-shadow` + `mix-blend-mode: screen/lighten` + `@property` animated conic gradients), SVG (`feGaussianBlur` + `feColorMatrix` threshold), and WebGL (emissive materials + `UnrealBloomPass`). The library tier produces the most filmic result; CSS produces the cheapest.
- **Particle-heavy** can be done at three scales: Canvas 2D `requestAnimationFrame` loops (thousands), tsParticles/Vanta drop-ins (tens of thousands), and WebGPU/WebGL compute or GPGPU (hundreds of thousands to a million).

---

## Details

# PILE 1 — VANILLA (the main pile)

### Scroll-driven & scroll choreography
- **Scroll-driven animations (`animation-timeline: scroll()`)** — Links any `@keyframes` animation to a scroll container's progress instead of time, running off the main thread. *Ref: MDN "CSS scroll-driven animations"; Bramus' scroll-driven-animations.style.* **Easy win.**
- **View-progress timelines (`animation-timeline: view()` + `animation-range`)** — Animates an element based on its own visibility as it crosses the viewport, perfect for reveal-on-scroll. *Ref: Codrops "A Practical Introduction to Scroll-Driven Animations"; WebKit blog.* **Easy win.**
- **Named view timelines (`view-timeline-name` / subscribing siblings)** — One element exposes a timeline that other (sticky) elements subscribe to, decoupling the tracked element from the animated one. *Ref: Josh W. Comeau "Scroll-Driven Animations."* **Medium.**
- **CSS-native parallax** — A `.parallax` wrapper sets `view-timeline-name` and children animate `translate` + `scale` off that timeline; no scroll events, GPU-accelerated. *Ref: CSS-Tricks "Bringing Back Parallax with Scroll-Driven CSS Animations"; dan-webnotes CSS-native parallax.* **Easy win.**
- **`position: sticky` choreography / scrollytelling** — Pin a "stage" with sticky so scrolling advances *time* inside a scene rather than moving it; combine with scroll timelines for grid reveals. *Ref: Codrops "Sticky Grid Scroll."* **Medium.**
- **Scroll-snap** — `scroll-snap-type` + `scroll-snap-align` create full-page or carousel snapping with no JS. **Easy win.**
- **Scroll progress reading indicator** — A fixed bar with `animation-timeline: scroll()` scaling X from 0→1. **Easy win.**

### Typed properties & gradient/color wizardry
- **`@property` typed custom properties** — Registering a variable with `syntax:'<angle>'`/`'<color>'`/`'<number>'` lets the browser *interpolate* it, unlocking true gradient/angle/hue animation in pure CSS. *Ref: MDN `@property`; Ryan Mulligan "CSS @property and the New Style."* **Medium — flag as high-wow.**
- **Animated conic-gradient glow border** — Animate a registered `--angle` 0→360deg inside `conic-gradient(from var(--angle), …)` painted on `border-box` via the padding-box/border-box double-background trick. *Ref: ibelick "animated gradient borders"; jhey on X.* **Medium — on-brand for neon.**
- **Hue-cycling with `@property --hue` into `hsl()`** — Animate a `<number>` and feed it to `hsl(var(--hue) …)` with staggered delays for color-wave grids. *Ref: Savvy "CSS @property guide."* **Easy win.**
- **`color-mix()`** — Mix two colors in a chosen color space directly in CSS for theme-able glows/tints. **Easy win.**
- **Gradient scroll (background-position)** — Oversized 200–400% gradient animated via `background-position` for the classic moving-gradient hero (Firefox-safe fallback for `@property`). *Ref: gradients.design.* **Easy win.**
- **`mix-blend-mode` / `backdrop-filter`** — `screen`/`lighten`/`plus-lighter` blend modes make overlapping glows add up; `backdrop-filter: blur()` powers glassmorphism. **Easy win.**

### Easing, motion path & transitions
- **`linear()` easing** — Arbitrary multi-stop easing (e.g., spring/bounce curves) expressed as a `linear()` function. **Medium.**
- **`cubic-bezier()` and `steps()`** — Custom curves and sprite-sheet/stepped animation. **Easy win.**
- **`offset-path` / motion path** — Move an element along an SVG `path()`/`ray()` by animating `offset-distance` 0→100%. *Ref: MDN "CSS motion path."* **Medium.**
- **View Transitions API (same-document)** — `document.startViewTransition(cb)` snapshots before/after DOM states and cross-fades (or custom-animates via `::view-transition-*` pseudo-elements). Baseline for same-document as of Firefox 144 (Oct 2025). *Ref: MDN "Using the View Transition API."* **Medium — high-wow.**
- **Cross-document View Transitions (MPA)** — Just add `@view-transition { navigation: auto; }` to both pages (same-origin) for animated page-to-page transitions with no JS; Chromium-only currently. *Ref: Chrome "Cross-document view transitions"; CSS-Tricks almanac.* **Medium — flag as least-known.**

### `:has()` & container-driven interactions
- **`:has()` relational selector** — Style/trigger animations on a parent based on descendant/sibling state (e.g., `.card:has(:hover)`), enabling pure-CSS interactive logic. **Medium.**
- **Container queries for responsive motion** — Animate/lay-out based on a container's size rather than the viewport. **Medium.**

### CSS masking, clipping & shapes
- **`clip-path` animation & morphing** — Animate `polygon()`/`inset()`/`circle()` with matching vertex counts for reveals, comparison sliders, and shape morphs; hardware-accelerated. *Ref: Emil Kowalski "The Magic of Clip Path"; MDN "Introduction to CSS clipping."* **Easy–Medium.**
- **`mask-image` + animated `mask-position`/`mask-size`** — Reveal/scan effects and gradient masks (e.g., flashlight reveals). *Ref: web.dev "Paths, shapes, clipping, masking."* **Medium.**
- **CSS anchor positioning** — Tether one element's position to an "anchor" element for tooltips/menus that track without JS. **Medium (newer).**

### SVG filter wizardry (organic / distortion / glow)
- **Gooey filter (`feGaussianBlur` + `feColorMatrix` threshold + `feComposite`)** — Blur merges nearby shapes, then an alpha-contrast matrix (`… 18 -7` / `19 -9` / `20 -7`) snaps the blur to a hard merged silhouette; classic blobby liquid menus, loaders, cursors. *Ref: Codrops "Creative Gooey Effects" (Lucas Bebber); Codrops "Gooey Cursor Effect."* **Medium — flag as high-wow, on-brand.**
- **`feTurbulence` (Perlin/fractal noise)** — Synthesizes clouds/marble/grain textures procedurally; `type="fractalNoise"` for smooth, `turbulence` for sharp. *Ref: Codrops "Creating Texture with feTurbulence"; MDN.* **Medium.**
- **`feTurbulence` + `feDisplacementMap`** — Noise displaces another image's pixels by channel for water/liquid/wavy distortion and "boiling line" hand-drawn motion; animate `baseFrequency`/`scale`. *Ref: Codrops; Camillo Visini "Simulating Hand-Drawn Motion"; MDN `feDisplacementMap`.* **Medium — high-wow.**
- **SVG drop-shadow / glow (`feGaussianBlur` on SourceAlpha + `feColorMatrix` + `feMerge`)** — Custom colored glows beyond `box-shadow`. *Ref: Codrops "SVG Filters 101" (Sara Soueidan).* **Medium.**
- **`feColorMatrix` hue-rotate / saturate / luminanceToAlpha** — Recolor, duotone, or convert luminance to alpha for masks. *Ref: W3C SVG 1.1 Filter Effects.* **Medium.**
- **SMIL animation (`<animate>`, `<set>`)** — Native SVG attribute animation (e.g., animate `baseFrequency`) without JS or CSS. **Medium.**
- **Animated `stroke-dashoffset` (line drawing)** — Animate dash offset on a path to "draw" SVG outlines/signatures. **Easy win.**
- **Path morphing** — Interpolate `d` attributes between compatible paths. **Medium.**
- **`foreignObject`** — Embed HTML inside SVG so filters apply to live DOM/images. **Medium.**

### Canvas 2D & generative graphics
- **`requestAnimationFrame` particle systems** — Thousands of particles via arrays + 2D context, the workhorse for dark particle fields. **Easy–Medium.**
- **Generative/noise art** — Perlin/simplex noise driving flow fields, blobs, and organic motion. **Medium.**
- **`ImageData` pixel manipulation** — Direct RGBA buffer access for dithering, glitch, and image effects. **Medium.**
- **`OffscreenCanvas` + Web Workers** — Render off the main thread for jank-free heavy canvas work. **Advanced.**
- **Canvas blend modes & text effects** — `globalCompositeOperation` (`lighter` for additive glow), gradient/stroked text. **Easy win.**

### Text effects
- **Text scramble / decode** — Randomly cycle characters then resolve to final text (the "hacker decode" look). *Ref: soulwire's CodePen "Text Scramble Effect"; Cruip tutorial.* **Easy win — on-brand.**
- **Variable font animation** — Animate `font-variation-settings` axes (`wght`, `wdth`, `slnt`) for kinetic type without multiple font files. *Ref: Mandy Michael's variable-font CodePens.* **Medium.**
- **Typewriter** — Reveal text char-by-char via `steps()` or JS. **Easy win.**
- **`text-wrap: balance` / `pretty`** — Balanced headline line-breaking. **Easy win.**
- **SVG-filter text distortion** — Apply `feTurbulence`+`feDisplacementMap` to headings while keeping text selectable. *Ref: henry.codes "How to distort text with SVG."* **Medium.**

### Cursor, pointer & micro-interactions
- **Custom cursor** — Hide native cursor; animate a `position: fixed` dot via rAF lerp toward pointer. *Ref: Codrops "Custom Cursor Effects" (Stefan Kaltenegger).* **Easy–Medium.**
- **Magnetic buttons** — Translate a button/element toward the pointer within a radius for a "magnetic" pull. *Ref: Codrops "Magnetic Buttons" (inspired by Cuberto).* **Medium — on-brand.**
- **Pointer/parallax tilt** — Map pointer position to `rotateX/rotateY` (3D tilt) or layered `translate`. **Easy–Medium.**
- **Gooey cursor trail** — Many small boxes + SVG gooey filter + blend mode following the cursor. *Ref: Codrops "Gooey Cursor Effect."* **Medium — high-wow.**

### 3D transforms & glass
- **CSS 3D transforms + `perspective`** — `preserve-3d`, `rotateX/Y/Z`, `translateZ` for card flips, cubes, and multi-layer parallax depth. **Medium.**
- **Glassmorphism** — `backdrop-filter: blur()` + translucent background + subtle border. **Easy win.**
- **Noise/grain overlay** — A tiled `feTurbulence` PNG/SVG at low opacity over the page for filmic texture. **Easy win — on-brand.**

### Web APIs for feel/polish
- **Web Animations API (`element.animate()`)** — Imperative JS keyframe animations sharing the CSS animation engine, composable and promise-based. *Ref: MDN.* **Medium.**
- **Intersection Observer** — Efficient reveal-on-scroll / lazy triggers without scroll listeners. **Easy win.**
- **Web Audio API (`AnalyserNode`)** — `getByteFrequencyData()`/`getByteTimeDomainData()` (FFT) drives audio-reactive Canvas/WebGL visuals (bars, radial, waveform). *Ref: MDN "Visualizations with Web Audio API"; Codrops 3D audio visualizer.* **Medium–Advanced — on-brand.**
- **`prefers-reduced-motion`** — Gate or tone down motion for accessibility. **Easy win — required.**
- **Vibration API / haptics** — `navigator.vibrate()` for tactile feedback on supported devices. **Easy win.**
- **Favicon / title animation, view-source easter eggs** — Animate the tab favicon/`document.title`; hide ASCII art in source. **Easy win.**

### Houdini
- **CSS Paint API (paint worklets)** — JS `paint()` worklets draw canvas-like backgrounds/borders/masks off the main thread, themeable via custom properties (generative patterns, blobs, confetti). *Ref: web.dev "houdini.how" (Una Kravets); George Francis' generative worklets; MDN CSS Painting API.* **Advanced — high-wow.**

### Cutting-edge built-in GPU
- **Raw WebGL + GLSL fragment shaders** — Direct GPU rendering into `<canvas>`; full-screen fragment shaders for procedural visuals. *Ref: MDN WebGPU/WebGL; The Book of Shaders.* **Advanced.**
- **Render-to-texture / ping-pong FBO feedback** — Two framebuffers alternate as input/output each frame for flowmaps, fluid, trails. *Ref: curtains.js flowmap examples.* **Advanced — high-wow.**
- **WebGPU + compute shaders (WGSL)** — Native compute pipeline with storage buffers runs particle physics fully on GPU (hundreds of thousands–1M particles). Google announced the cross-browser milestone November 25, 2025; per webgpu.com, "as of November 2025, WebGPU ships by default in Chrome, Firefox, Safari, and Edge" (Chrome since v113/2023; Safari 26.0 Sept 2025; Firefox 141 on Windows July 2025, Firefox 145 on macOS ARM64). *Ref: MDN WebGPU API; Three.js Roadmap "Interactive Galaxy with WebGPU Compute Shaders"; lisyarus "Particle Life in WebGPU."* **Advanced — flag as least-known frontier.**

---

# PILE 2 — FRAMEWORKS & LIBRARIES (secondary, wider net)

### Animation engines & sequencing
- **GSAP (GreenSock)** — Industry-standard high-speed property animator with precise timelines. It became 100% free — including all former Club plugins (SplitText, MorphSVG, DrawSVG, ScrollTrigger, ScrollSmoother) and commercial use — on April 30, 2025 with the v3.13 release, following Webflow's October 2024 acquisition; Webflow's blog states "everyone…will be able to leverage all of GSAP's tools completely free of charge, including the previously paid Club plugins." *Ref: gsap.com; Webflow blog.*
- **GSAP ScrollTrigger** — The standard for scroll-driven sequencing, pinning, scrubbing, and snap. *Ref: gsap.com/scroll.*
- **GSAP ScrollSmoother** — Smooth/momentum scrolling on top of ScrollTrigger.
- **GSAP Flip** — Animate between two layout states (First-Last-Invert-Play) for list/grid reflows.
- **GSAP SplitText** — Split text into lines/words/chars for staggered animation (rebuilt with masking + auto-resize).
- **GSAP MorphSVG** — Fluidly morph one SVG path into another even with differing point counts.
- **GSAP MotionPath** — Animate elements along an SVG path.
- **GSAP DrawSVG** — Animated SVG stroke drawing.
- **GSAP Observer** — Unified cross-device gesture/scroll/pointer event normalization.
- **GSAP Physics2D / Inertia** — Velocity/gravity-driven motion (e.g., letters shattering off-screen). *Ref: Codrops "From SplitText to MorphSVG."*
- **Motion (formerly Framer Motion / Motion One)** — Modern animation library; the only one running scroll-linked animations on native `ScrollTimeline` where possible; great React fit (~8KB core). *Ref: motion.dev.*
- **Anime.js (v4)** — Lightweight modular JS animation engine with a physics/spring system. *Ref: cssauthor 2026 roundup.*
- **Mo.js / Popmotion** — Motion-graphics/burst animation toolkits.
- **Theatre.js** — Cinematic sequence editor with a visual timeline/graph for keyframing Three.js or DOM (bridges code + GUI). *Ref: theatrejs.com.*

### Smooth scroll & page transitions
- **Lenis (darkroom.engineering)** — The leading <4KB smooth-scroll library; wraps native scroll so `position: sticky`, anchors, and accessibility keep working; built to sync WebGL + DOM and GSAP ScrollTrigger. *Ref: lenis.dev; GitHub darkroomengineering/lenis.*
- **Locomotive Scroll** — Smooth scroll + parallax via transformed container; now largely superseded by Lenis (it alters DOM origin). *Ref: borndigital.be comparison.*
- **ScrollMagic** — Older scene-based scroll library, effectively obsolete vs Intersection Observer.
- **Barba.js** — ~9KB AJAX + lifecycle-hook page transitions for SPA-like MPA navigation (pair with GSAP). *Ref: barba.js.org; Codrops Astro+Barba tutorial.*
- **Swup** — Drop-in CSS-class-driven page transitions with cache/preload. *Ref: swup.js.org.*
- **Highway.js** — Lightweight page-transition manager.

### 3D / WebGL / WebGPU
- **Three.js** — The dominant general-purpose JS 3D library (scene graph, materials, loaders, renderer); created by Ricardo Cabello (Mr.doob) and first released April 2010. It is by far the most-used 3D library, reaching ~2.7M weekly npm downloads by March 2026 ("270x more than Babylon.js," ~10K) per utsubo.com "What's New in Three.js (2026)" and technologychecker.io. *Ref: threejs.org; utsubo.com; technologychecker.io.*
- **React Three Fiber (R3F)** — Declarative React renderer for Three.js (JSX maps to Three objects), by pmndrs. *Ref: r3f.docs.pmnd.rs.*
- **drei (@react-three/drei)** — Huge helper library for R3F: controls, loaders, `<Environment>`/HDRI, SDF text, staging, shadows, HTML overlays. *Ref: github.com/pmndrs/drei.*
- **Babylon.js** — Full batteries-included 3D game engine (physics, PBR, GUI, inspector, node material editor, strong WebXR), Microsoft-backed. *Ref: babylonjs.com.*
- **PlayCanvas** — Unity-style editor-driven, collaborative cloud engine with tiny fast runtime and strong WebGPU; used by Snap/BMW/Disney. *Ref: playcanvas.com.*
- **OGL** — Minimal ~8KB WebGL library for developers who want to write their own shaders close to native WebGL. *Ref: github.com/oframe/ogl.*
- **regl** — Stateless functional WebGL abstraction (resources + commands compiled to optimized JS). *Ref: regl-project.github.io/regl.*
- **curtains.js** — Vanilla WebGL library that turns DOM images/videos into shader-animated textured planes synced to CSS layout (displacement, flowmaps, post-processing). *Ref: curtainsjs.com (Martin Laxenaire); CSS-Tricks "Creating WebGL Effects with CurtainsJS."*
- **PixiJS** — Fastest, most popular 2D WebGL/WebGPU renderer ("HTML5 Creation Engine"); v8 integrates WebGPU. *Ref: pixijs.com.*

### Creative coding & vector
- **p5.js** — Beginner-friendly creative-coding library (Processing for the web) with `setup()`/`draw()`. *Ref: p5js.org.*
- **Paper.js** — Vector graphics scripting framework on Canvas with retained scene graph and Bézier/boolean ops. *Ref: paperjs.org.*
- **Two.js** — Renderer-agnostic (SVG/Canvas/WebGL) 2D drawing + animation API. *Ref: two.js.org.*

### Shader/creative tooling
- **Shadertoy** — Community + live-coding platform for GLSL fragment shaders (no vertex shader); by Iñigo Quilez & Pol Jeremias, online since 2013. *Ref: shadertoy.com.*
- **Raymarching SDFs** — Render 3D/volumetric scenes in a single fragment shader by stepping rays via signed-distance functions; soft shadows and glow come cheaply, and density accumulation yields volumetric glow. *Ref: Iñigo Quilez iquilezles.org; Jamie Wong "Ray Marching and SDFs."*
- **TSL (Three.js Shading Language)** — Write shaders in JS that transpile to GLSL or WGSL, future-proofing material code for WebGPU. *Ref: Maxime Heckel "Field Guide to TSL and WebGPU."*
- **glslify** — Module system for composing GLSL shaders.
- **Spector.js** — WebGL frame-debugging/capture tool.

### Vector/motion runtimes
- **Lottie** — Renders After Effects animations exported as JSON; great for linear marketing/UI motion. It is the most widely adopted animation format in production software, with 4.2M weekly npm downloads across lottie-web/-ios/-android per LottieFiles' "LottieFiles or Rive" blog. *Ref: lottiefiles.com.*
- **Rive** — Interactive, state-machine-driven vector animation with a tiny GPU-accelerated `.riv` binary. Per Rive's official "Rive vs Lottie," its files are "typically 10-15x smaller than equivalent Lottie files" (e.g., a 240KB Lottie recreated as 16KB). Rive states Spotify Wrapped and LinkedIn Year in Review launched with Rive, and that "at the end of 2025, Rive went out to ~1.7B end users"; Duolingo reported a 15x file-size reduction adopting it. *Ref: rive.app.*
- **Spline** — Browser 3D design tool with easy embed (popular with Framer). *Ref: framer.com web-animation-tools.*

### Effect-specific
- **tsParticles** — TypeScript particle engine for configurable particle/confetti/firework backgrounds (successor to particles.js); line-linked dots, starfields, snow. *Ref: particles.js.org.*
- **Vanta.js** — Ready-to-use animated 3D/WebGL backgrounds (NET constellations, birds, waves, fog, globe) built on Three.js/p5. *Ref: vantajs.com.*
- **Matter.js / Rapier** — 2D physics engines (Rapier is Rust/WASM, faster).
- **typed.js** — Typewriter text effect library.
- **Splitting.js** — Splits text/elements for CSS-variable-driven staggered animation.
- **Tween.js** — Lightweight tweening engine.
- **Blotter.js** — GLSL-backed creative text effects.

### Famous studios & reference sites to study
- **Bruno Simon** (bruno-simon.com) — Iconic 3D driving-game portfolio; Three.js + physics + matcaps + TSL/WebGPU; creator of the Three.js Journey course. *Ref: Awwwards SOTD case study.*
- **Lusion** (lusion.co) — High-end WebGL/interactive studio (Lusion v3 Awwwards SOTD).
- **Active Theory** — Immersive WebGL experiences (e.g., Spotify "Listening Together").
- **Resn**, **Dogstudio**, **Immersive Garden** — Award-winning creative-dev studios.
- **Codrops (tympanus.net)** — The canonical tutorial source for nearly every effect above.
- **Awwwards "Site of the Day"** — Pattern reference for the genre.
- **Stripe / Apple product pages / Linear** — Mainstream exemplars of restrained, high-craft scroll and gradient polish.
- **luruke/awesome-casestudy** (GitHub) — Curated technical WebGL case studies.

---

## Recommendations

**Stage 1 — Build the gallery shell vanilla (week 1).** Use scroll-driven animations (`scroll()`/`view()`) for all reveals and parallax, `@property` conic gradients for neon borders, `color-mix`/`hsl(@property --hue)` for the cyan/coral/teal palette, View Transitions for navigation between effect pages, and `prefers-reduced-motion` from day one. These are easy wins that cost nothing in bundle size. *Benchmark to advance: if you need precise multi-step orchestration or cross-browser scroll consistency (Safari/Firefox gaps in scroll-driven animations), add GSAP ScrollTrigger + Lenis.*

**Stage 2 — Add the signature "wow" vanilla effects (week 2).** Ship the SVG gooey filter (blob menus/cursor), `feTurbulence`+`feDisplacementMap` distortion on hero imagery, a Canvas 2D particle field, a text-scramble headline, and a Web Audio `AnalyserNode` visualizer for an audio-reactive centerpiece. These are medium-difficulty and define the brand. *Benchmark to advance: if particle counts exceed ~10–20k or you want true volumetric glow, move to WebGL/WebGPU.*

**Stage 3 — Library tier for the cinematic ceiling (week 3+).** Introduce Three.js (or R3F + drei) with `UnrealBloomPass`/`@react-three/postprocessing` for the filmic glow on dark, curtains.js or OGL for WebGL-on-DOM image distortion, and Rive/Lottie for any vector mascot/UI motion. Study Shadertoy raymarched-SDF shaders for volumetric glow. *Benchmark: only adopt WebGPU compute particles if you specifically need >100k animated particles; otherwise the cost/complexity isn't justified.*

**Cross-cutting:** Keep filters inline near their art with unique IDs (SVG filter URL references are fragile across shadow roots), wrap decorative filtered regions in `aria-hidden`, and prefer compositor-friendly properties (`transform`, `opacity`, `translate`, `scale`) to avoid INP/jank.

---

## Caveats
- **Browser support is uneven at the frontier.** Scroll-driven animations are Chromium/WebKit with Firefox behind a flag in places; cross-document View Transitions are Chromium-only; `@property` historically lagged in Firefox. Always feature-detect (`@supports`) and provide fallbacks (e.g., `background-position` gradients for `@property`).
- **SVG filters are GPU/CPU-intensive** — keep animated filter regions small; the gooey filter creates a stacking context that can trap clicks and break form layouts beneath it.
- **Some 2026 figures are time-sensitive or secondhand** — npm download counts (Three.js ~2.7M vs Babylon ~10K weekly per utsubo.com/technologychecker.io; Lottie 4.2M per LottieFiles), bundle sizes, and "industry standard" claims come from blog roundups and vendor sources and should be treated as approximate. The Webflow→GSAP "100% free" change (v3.13, Apr 30 2025) and WebGPU shipping by default in all major browsers (announced Nov 25, 2025) are well-corroborated.
- **"Less than 1% of websites" is a qualitative framing, not a measured statistic** — it reflects the rarity of polished execution, not a verified metric.
- This catalog prioritizes breadth; each technique's production depth (cross-browser edge cases, performance tuning) requires the linked references.

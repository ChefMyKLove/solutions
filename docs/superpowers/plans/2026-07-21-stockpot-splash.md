# The Stockpot Splash — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace `index.html` (currently the services ticket) with an animated,
hand-rolled Canvas 2D "stockpot boil" splash that resolves into four bubble
nav buttons (Menu / Chef / Pass / Front of House), each expanding in place
into a full content panel; move the current ticket to `menu.html`.

**Architecture:** Single-file, dependency-free `index.html` (inline CSS + JS,
matching the repo's existing convention). Canvas 2D is used **only** for the
decorative pre-boil spectacle (pot silhouette, burner glow, rising/popping
ambient bubbles) — it never carries interactive or accessible content. The
four real navigation bubbles are ordinary `<button>` elements, CSS-styled to
look like bubbles, positioned at fixed resting coordinates, animated only
with CSS (ambient bob + hover/focus wobble). The "four bubbles break the
surface and become the nav" moment is achieved with a **timed crossfade**:
the canvas/pot fades out while the DOM bubble buttons fade + scale in at the
same moment — not a literal handoff of one simulated object to another. This
keeps 100% of interactive surface in the accessibility tree while keeping
100% of the boil spectacle in canvas, and is simple enough to build, debug,
and read in view-source.

**Tech Stack:** Plain HTML5, CSS3, vanilla JavaScript (ES2020+), Canvas 2D.
No build step, no npm, no frameworks, no external JS libraries. Fonts via
existing Google Fonts `<link>` pattern. Forms submit to Web3Forms via
`fetch`.

## Global Constraints

- **Zero external JS dependencies** — no libraries, no bundler, no build
  step. (Spec §2, §3)
- Palette reuses existing CSS custom properties: `--paper`, `--paper-shadow`,
  `--char`, `--ember`, `--sage`, `--slate`, `--line`. No competing palette
  introduced. (Spec §4)
- Boil sequence plays **once per browser session** via `sessionStorage`; any
  click/tap/keypress during the boil skips straight to resolved state; a
  visible **"Boil Again ↻"** control lets the visitor manually replay it at
  any time afterward, without touching the session-skip behavior. (Spec §5,
  and 2026-07-21 spec addendum)
- `prefers-reduced-motion: reduce` disables the boil animation and the
  ambient drift entirely — bubbles render statically and remain fully
  functional; "Boil Again" is hidden/replaced with a static acknowledgment
  under this setting. (Spec §5 and addendum)
- The four bubble buttons are real, always-present `<button>` elements in
  the DOM; the canvas is `aria-hidden="true"` and purely decorative. Nothing
  interactive ever lives only in canvas. (Spec §5)
- Only one content panel open at a time; `Esc` and a visible `×` both close
  the open panel back to the resolved bubble state (never re-running the
  boil). (Spec §6)
- No routing, no hash URLs, no history entries, no analytics, no GitHub
  links in The Pass, no literal Kitchen-Confidential cover pastiche — the
  homage is palette + type attitude + first-person voice only. (Spec §9)
- Every factual/bracketed placeholder in copy (`[LIKE THIS]`) is an
  intentional author fill-in flagged by the spec's §10 — these are not plan
  defects, do not resolve them with invented facts.
- **Testing approach for this codebase:** there is no test runner, package
  manager, or build step in this static site, so steps use two verification
  styles instead of a unit-test framework: (a) **Manual Browser
  Verification** — concrete, observable checks performed by opening the
  file in a real browser (serve it locally: `python -m http.server 8000` or
  `npx serve`, since `fetch()` and `sessionStorage` behave inconsistently
  under `file://`); and (b) **Scratch Node Verification** — for pure math
  helper functions only, temporarily copy the function into a throwaway
  file, run it with plain `node`, confirm the printed assertions, then
  delete the throwaway file. Every step spells out exactly what to check.

---

## File Structure

- **Modify → Create:** `menu.html` — the current `index.html` content moved
  here verbatim, plus one added "← back to the boil" link.
- **Rewrite:** `index.html` — the new splash: stage (canvas + masthead +
  bubble nav + Boil Again control) and four overlay panels (Menu, Chef,
  Pass, Front of House), all inline CSS/JS.

No other files. No `package.json`, no `node_modules`, no bundler config.

---

### Task 1: Relocate the current ticket to `menu.html`

**Files:**
- Create: `menu.html` (copy of current `index.html`, then modified)
- Modify: none yet (current `index.html` is rewritten wholesale in Task 2
  onward; this task only produces `menu.html`)

**Interfaces:**
- Produces: a standalone, printable ticket page at `menu.html` that later
  tasks link to from the splash's Menu panel via `href="menu.html"`.

- [ ] **Step 1: Copy the current `index.html` to `menu.html`**

```bash
cp "index.html" "menu.html"
```

- [ ] **Step 2: Add a "back to the boil" link and update the title/og tags**

In `menu.html`, change the `<title>` and add a back-link right after the
opening `<body>` tag (before `<div class="ticket">`):

```html
<title>Menu — Chefmyklove Custom Software Solutions</title>
```

```html
<body>

<a href="index.html" style="display:block; text-align:center; font-family:'Space Mono', monospace; font-size:12px; letter-spacing:1px; color:#5C6B54; text-decoration:none; padding-top:12px;">&larr; back to the boil</a>

<div class="ticket">
```

- [ ] **Step 3: Manual Browser Verification**

Run: `python -m http.server 8000` from the project root, then open
`http://localhost:8000/menu.html`.

Expected: the ticket renders exactly as before, the QR code still points to
`solutions.chefmyklove.com`, the intake form is present, and a "← back to
the boil" link appears above the ticket and currently 404s (that's fine —
`index.html` gets rewritten in Task 2). Also check `@media print` still
hides the intake form when printing (browser print preview).

- [ ] **Step 4: Commit**

```bash
git add menu.html
git commit -m "Move services ticket to menu.html ahead of new splash"
```

---

### Task 2: New `index.html` shell — stage, masthead, resting bubble layout

**Files:**
- Rewrite: `index.html`

**Interfaces:**
- Produces: the page shell with `#stage`, `#boil-canvas`, `#masthead`,
  `#bubble-nav` containing four `<button class="bubble" data-panel="...">`
  elements at fixed CSS resting positions, and a hidden `#boil-again`
  button. Establishes CSS custom properties and font loading that every
  later task builds on.
- Consumes: nothing (first task to touch this file).

- [ ] **Step 1: Write the new `index.html` shell**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Chefmyklove — Custom Software, Cooked to Order</title>
<meta property="og:title" content="Chefmyklove — Custom Software, Cooked to Order">
<meta property="og:description" content="Custom software from a guy who used to work the line.">
<meta property="og:type" content="website">
<meta property="og:url" content="https://solutions.chefmyklove.com">
<meta name="twitter:card" content="summary">
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Archivo+Black&family=Space+Mono:wght@400;700&family=Work+Sans:wght@400;500;600&display=swap" rel="stylesheet">
<!--
  This page is hand-rolled: no frameworks, no build step, no third-party
  JS. The boil is Canvas 2D driven by plain functions you can read start
  to finish below. Poke around — that's kind of the point.
-->
<style>
:root{
  --paper: #F7F3E9;
  --paper-shadow: #EAE3D3;
  --char: #1F1B16;
  --ember: #B8452E;
  --sage: #5C6B54;
  --slate: #333F3A;
  --line: #C9C0AA;
  --stage-dark: #14110D;
}
*{ box-sizing:border-box; margin:0; padding:0; }
html, body{ height:100%; }
body{
  background: var(--stage-dark);
  font-family:'Work Sans', sans-serif;
  color: var(--char);
  overflow:hidden;
}
#stage{
  position:relative;
  width:100vw;
  height:100vh;
  background: var(--stage-dark);
  transition: background-color 1.1s ease;
}
#stage.resolved{ background: var(--paper-shadow); }
#boil-canvas{
  position:absolute;
  inset:0;
  width:100%;
  height:100%;
  transition: opacity 0.9s ease;
}
#stage.resolved #boil-canvas{ opacity:0; pointer-events:none; }

#masthead{
  position:absolute;
  top:12%;
  left:50%;
  transform: translateX(-50%);
  text-align:center;
  color: var(--paper);
  opacity:0;
  transition: opacity 0.9s ease 0.3s, color 0.9s ease 0.3s;
  width:90%;
  max-width:640px;
}
#stage.resolved #masthead{ opacity:1; color: var(--char); }
#masthead h1{
  font-family:'Archivo Black', 'Space Mono', monospace;
  font-size: clamp(32px, 7vw, 56px);
  letter-spacing: -1px;
  color: var(--ember);
  text-transform: uppercase;
}
#masthead p{
  margin-top: 10px;
  font-family:'Space Mono', monospace;
  font-size: clamp(12px, 2vw, 15px);
  letter-spacing: 0.5px;
}

#bubble-nav{
  position:absolute;
  inset:0;
}
.bubble{
  position:absolute;
  width: clamp(120px, 16vw, 170px);
  height: clamp(120px, 16vw, 170px);
  border-radius:50%;
  background: var(--paper);
  border: 2px solid var(--ember);
  color: var(--char);
  font-family:'Space Mono', monospace;
  font-size: 13px;
  font-weight:700;
  letter-spacing: 1px;
  text-transform: uppercase;
  cursor:pointer;
  opacity:0;
  transform: translate(-50%, -50%) scale(0.3);
  transition: opacity 0.7s ease, transform 0.7s ease;
  box-shadow: 0 10px 24px rgba(0,0,0,0.18);
}
#stage.resolved .bubble{ opacity:1; transform: translate(-50%, -50%) scale(1); }
.bubble:hover, .bubble:focus-visible{
  transform: translate(-50%, -50%) scale(1.08) rotate(-3deg);
  outline: 3px solid var(--sage);
  outline-offset: 3px;
}
.bubble[data-panel="menu"]{ top: 32%; left: 30%; }
.bubble[data-panel="chef"]{ top: 58%; left: 66%; }
.bubble[data-panel="pass"]{ top: 66%; left: 30%; }
.bubble[data-panel="foh"]{  top: 30%; left: 68%; }

#boil-again{
  position:absolute;
  bottom: 24px;
  left:50%;
  transform: translateX(-50%);
  font-family:'Space Mono', monospace;
  font-size: 12px;
  letter-spacing: 1px;
  background: transparent;
  border: 1px solid var(--slate);
  color: var(--char);
  padding: 8px 16px;
  border-radius: 20px;
  cursor:pointer;
  opacity:0;
  transition: opacity 0.6s ease;
}
#stage.resolved #boil-again{ opacity:0.8; }
#boil-again:hover{ opacity:1; }

@media (max-width: 700px){
  #bubble-nav{
    position:absolute;
    inset: 20% 8% 12%;
    display:grid;
    grid-template-columns: 1fr 1fr;
    grid-template-rows: 1fr 1fr;
    gap: 16px;
    place-items:center;
  }
  .bubble{ position:static; transform: scale(0.3); }
  #stage.resolved .bubble{ transform: scale(1); }
  .bubble:hover, .bubble:focus-visible{ transform: scale(1.05); }
}
</style>
</head>
<body>

<div id="stage">
  <canvas id="boil-canvas" aria-hidden="true"></canvas>

  <header id="masthead">
    <h1>Chefmyklove</h1>
    <p id="tagline">Custom software from a guy who used to work the line.</p>
  </header>

  <nav id="bubble-nav" aria-label="Site sections">
    <button class="bubble" data-panel="menu">The Menu</button>
    <button class="bubble" data-panel="chef">The Chef</button>
    <button class="bubble" data-panel="pass">The Pass</button>
    <button class="bubble" data-panel="foh">Front of House</button>
  </nav>

  <button id="boil-again" type="button">Boil Again &#8635;</button>
</div>

<script>
  // Force-resolved for now so we can see the resting layout before the
  // boil engine exists (Task 5 removes this line).
  document.getElementById('stage').classList.add('resolved');
</script>

</body>
</html>
```

- [ ] **Step 2: Manual Browser Verification**

Run: `python -m http.server 8000` (if not already running), open
`http://localhost:8000/index.html`.

Expected: dark-to-paper background switch happens immediately (because of
the temporary force-resolve script), the "Chefmyklove" headline and tagline
appear in ember red on cream, and four paper-colored circular buttons
labeled THE MENU / THE CHEF / THE PASS / FRONT OF HOUSE are visible at
their four distinct screen positions without overlapping the masthead.
Resize the window below 700px width and confirm the four bubbles switch to
a 2×2 grid layout. No console errors.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Add new splash shell with resting bubble-nav layout"
```

---

### Task 3: Ambient bob + hover/focus wobble for resting bubbles

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `.bubble` elements from Task 2.
- Produces: a `applyAmbientMotion()` function, called once on load, that
  gives each bubble randomized independent bobbing via CSS custom
  properties (`--dx`, `--dy`, `--rot`, `--dur`, `--delay`), respecting
  `prefers-reduced-motion`.

- [ ] **Step 1: Add the bob keyframes and custom-property hooks to the `<style>` block**

Insert after the `.bubble` rules:

```css
@keyframes bob{
  0%, 100%{ transform: translate(-50%, -50%) translate(0, 0) rotate(0deg); }
  50%{ transform: translate(-50%, -50%) translate(var(--dx, 6px), var(--dy, -8px)) rotate(var(--rot, 2deg)); }
}
.bubble.ambient{
  animation: bob var(--dur, 5s) ease-in-out var(--delay, 0s) infinite;
}
@media (prefers-reduced-motion: reduce){
  .bubble.ambient{ animation: none; }
}
```

- [ ] **Step 2: Add the ambient-motion script**

Add before the closing `</script>` of the existing script block (replacing
nothing yet — this is additive):

```html
<script>
  function applyAmbientMotion(){
    var reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    var bubbles = document.querySelectorAll('.bubble');
    bubbles.forEach(function(bubble, i){
      if (reduce) return;
      var dx = (Math.random() * 16 - 8).toFixed(1) + 'px';
      var dy = (Math.random() * 16 - 10).toFixed(1) + 'px';
      var rot = (Math.random() * 6 - 3).toFixed(1) + 'deg';
      var dur = (4 + Math.random() * 3).toFixed(2) + 's';
      var delay = (i * 0.4 + Math.random() * 0.6).toFixed(2) + 's';
      bubble.style.setProperty('--dx', dx);
      bubble.style.setProperty('--dy', dy);
      bubble.style.setProperty('--rot', rot);
      bubble.style.setProperty('--dur', dur);
      bubble.style.setProperty('--delay', delay);
      bubble.classList.add('ambient');
    });
  }
  applyAmbientMotion();
</script>
```

- [ ] **Step 3: Manual Browser Verification**

Reload `http://localhost:8000/index.html`.

Expected: each of the four bubbles drifts slowly and independently (not in
lockstep — different timing per bubble). Hover over one: it scales up and
tilts slightly, showing a visible sage-colored focus-style outline. Tab
through the page with keyboard only: each bubble receives the same visible
outline in sequence (Menu, Chef, Pass, Front of House) and nothing else on
the page is focusable yet. In Chrome DevTools, toggle "Emulate CSS
prefers-reduced-motion: reduce" (Rendering tab) and reload: bubbles must
sit still with no bobbing.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add ambient bob and hover/focus wobble to bubble nav"
```

---

### Task 4: Canvas boil engine (pure functions + render loop)

**Files:**
- Modify: `index.html`

**Interfaces:**
- Produces: pure functions `mulberry32(seed)`, `createAmbientBubble(rng, w, h)`,
  `stepAmbientBubble(b, dt)` — usable standalone and verifiable with Node —
  plus a `startBoilRender(canvas)` function that runs a `requestAnimationFrame`
  loop drawing a pot silhouette, a burner glow, and a population of ambient
  bubbles using those pure functions. Does **not** yet trigger the
  boil→resolve transition (Task 5) or hook into session/skip/reduced-motion
  logic (Task 5) — this task only makes the canvas draw convincingly when
  manually started.
- Consumes: `#boil-canvas` element from Task 2.

- [ ] **Step 1: Add the pure math/physics functions**

Add a new `<script>` block, inserted **before** the existing ambient-motion
script block:

```html
<script>
  // ---- Pure functions: seeded RNG + ambient bubble physics ----
  // No DOM access in this block on purpose - see scratch verification below.

  function mulberry32(seed) {
    return function() {
      seed |= 0; seed = (seed + 0x6D2B79F5) | 0;
      var t = Math.imul(seed ^ (seed >>> 15), 1 | seed);
      t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
      return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
    };
  }

  function createAmbientBubble(rng, w, h) {
    return {
      x: w / 2 + (rng() - 0.5) * w * 0.5,
      y: h - 6,
      r: 2 + rng() * 4,
      vy: -(24 + rng() * 30),
      wobblePhase: rng() * Math.PI * 2,
      wobbleSpeed: 1 + rng() * 2,
      popAtY: h * 0.16 + rng() * h * 0.10,
      popped: false
    };
  }

  function stepAmbientBubble(b, dt) {
    var y = b.y + b.vy * dt;
    var wobblePhase = b.wobblePhase + b.wobbleSpeed * dt;
    var x = b.x + Math.sin(wobblePhase) * 10 * dt;
    var popped = y <= b.popAtY;
    return {
      x: x, y: y, r: b.r, vy: b.vy,
      wobblePhase: wobblePhase, wobbleSpeed: b.wobbleSpeed,
      popAtY: b.popAtY, popped: popped
    };
  }
</script>
```

- [ ] **Step 2: Scratch Node Verification of the pure functions**

Create a throwaway file `scratch-bubble-math.mjs` in the project root with:

```js
function mulberry32(seed) {
  return function() {
    seed |= 0; seed = (seed + 0x6D2B79F5) | 0;
    var t = Math.imul(seed ^ (seed >>> 15), 1 | seed);
    t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
    return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
  };
}
function createAmbientBubble(rng, w, h) {
  return {
    x: w / 2 + (rng() - 0.5) * w * 0.5, y: h - 6, r: 2 + rng() * 4,
    vy: -(24 + rng() * 30), wobblePhase: rng() * Math.PI * 2,
    wobbleSpeed: 1 + rng() * 2, popAtY: h * 0.16 + rng() * h * 0.10, popped: false
  };
}
function stepAmbientBubble(b, dt) {
  var y = b.y + b.vy * dt;
  var wobblePhase = b.wobblePhase + b.wobbleSpeed * dt;
  var x = b.x + Math.sin(wobblePhase) * 10 * dt;
  var popped = y <= b.popAtY;
  return { x: x, y: y, r: b.r, vy: b.vy, wobblePhase: wobblePhase,
    wobbleSpeed: b.wobbleSpeed, popAtY: b.popAtY, popped: popped };
}

var rng = mulberry32(42);
var b = createAmbientBubble(rng, 800, 600);
console.assert(b.y === 594, 'bubble should spawn near canvas bottom, got ' + b.y);
console.assert(b.vy < 0, 'bubble should move upward (negative vy)');

var steps = 0;
var current = b;
while (!current.popped && steps < 500) {
  current = stepAmbientBubble(current, 1 / 60);
  steps++;
}
console.assert(current.popped === true, 'bubble should eventually pop, popped=' + current.popped);
console.assert(steps < 500, 'bubble took too many steps to pop: ' + steps);
console.assert(current.y <= current.popAtY, 'bubble should pop at or above its popAtY line');

console.log('All bubble-math assertions passed. steps to pop: ' + steps);
```

Run: `node scratch-bubble-math.mjs`
Expected output: `All bubble-math assertions passed. steps to pop: <some number under 500>`
with no `Assertion failed` lines printed above it.

- [ ] **Step 3: Delete the scratch file**

```bash
rm scratch-bubble-math.mjs
```

- [ ] **Step 4: Add the canvas render loop (pot, burner, ambient bubbles)**

Add this script block right after the pure-functions block from Step 1:

```html
<script>
  function startBoilRender(canvas) {
    var ctx = canvas.getContext('2d');
    var dpr = window.devicePixelRatio || 1;
    var rng = mulberry32(Date.now() & 0xffffffff);
    var bubbles = [];
    var lastTime = null;
    var running = true;

    function resize() {
      canvas.width = window.innerWidth * dpr;
      canvas.height = window.innerHeight * dpr;
      canvas.style.width = window.innerWidth + 'px';
      canvas.style.height = window.innerHeight + 'px';
      ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    }
    resize();
    window.addEventListener('resize', resize);

    var maxBubbles = window.innerWidth < 700 ? 14 : 26;

    function spawnIfNeeded(w, h) {
      if (bubbles.length < maxBubbles) {
        bubbles.push(createAmbientBubble(rng, w, h));
      }
    }

    function drawPotAndBurner(w, h) {
      var potTop = h * 0.72;
      var glowRadius = Math.min(w, h) * 0.18;
      var glow = ctx.createRadialGradient(w / 2, h * 0.92, 4, w / 2, h * 0.92, glowRadius);
      glow.addColorStop(0, 'rgba(184,69,46,0.9)');
      glow.addColorStop(1, 'rgba(184,69,46,0)');
      ctx.fillStyle = glow;
      ctx.fillRect(0, h * 0.8, w, h * 0.2);

      ctx.fillStyle = '#0B0906';
      ctx.beginPath();
      ctx.moveTo(w * 0.28, h);
      ctx.lineTo(w * 0.32, potTop);
      ctx.lineTo(w * 0.68, potTop);
      ctx.lineTo(w * 0.72, h);
      ctx.closePath();
      ctx.fill();
    }

    function drawBubble(b) {
      ctx.beginPath();
      ctx.arc(b.x, b.y, b.r, 0, Math.PI * 2);
      ctx.strokeStyle = 'rgba(247,243,233,0.55)';
      ctx.lineWidth = 1;
      ctx.stroke();
    }

    function frame(t) {
      if (!running) return;
      var w = window.innerWidth, h = window.innerHeight;
      if (lastTime === null) lastTime = t;
      var dt = Math.min((t - lastTime) / 1000, 0.05);
      lastTime = t;

      ctx.clearRect(0, 0, w, h);
      drawPotAndBurner(w, h);

      spawnIfNeeded(w, h);
      bubbles = bubbles
        .map(function(b) { return stepAmbientBubble(b, dt); })
        .filter(function(b) { return !b.popped; });
      bubbles.forEach(drawBubble);

      requestAnimationFrame(frame);
    }
    requestAnimationFrame(frame);

    return {
      stop: function() { running = false; window.removeEventListener('resize', resize); }
    };
  }
</script>
```

- [ ] **Step 5: Temporarily start the render loop to verify it visually, then leave the call in place for Task 5 to take over**

Replace the placeholder script at the bottom of `<body>` (the one that
force-adds `.resolved`) with:

```html
<script>
  var boilHandle = startBoilRender(document.getElementById('boil-canvas'));
  applyAmbientMotion();
</script>
```

- [ ] **Step 6: Manual Browser Verification**

Reload `http://localhost:8000/index.html`.

Expected: the stage stays dark (no `.resolved` class yet, since we removed
the placeholder), a black pot silhouette sits in the lower-middle of the
screen with an ember-colored glow beneath it, and small pale bubbles
continuously rise out from the pot's rim area, wobbling side to side, and
disappear (pop) partway up the screen. Resize the browser window and
confirm the pot/bubbles redraw at the new size without stretching oddly.
Leave the page open 30+ seconds: confirm no runaway memory growth in
DevTools Performance/Memory (bubble count should stay capped, not grow
unbounded).

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "Add canvas boil engine: pot silhouette, burner glow, ambient bubbles"
```

---

### Task 5: Boil→resolve orchestration, session skip, reduced-motion, and click-to-skip

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `startBoilRender()` (Task 4), `#stage.resolved` CSS (Task 2),
  `.bubble.ambient` (Task 3).
- Produces: `resolveStage()` function that stops the boil render and adds
  `.resolved` to `#stage`; a `runBoilSequence()` function that either runs
  the full timed boil or skips straight to resolved, based on
  `sessionStorage.getItem('boiled')` and `prefers-reduced-motion`; a
  document-level listener that skips the boil immediately on the first
  click/keypress/touch while it's running.

- [ ] **Step 1: Replace the Step-5 bootstrap script from Task 4 with the full orchestration**

Replace:

```html
<script>
  var boilHandle = startBoilRender(document.getElementById('boil-canvas'));
  applyAmbientMotion();
</script>
```

with:

```html
<script>
  var BOIL_DURATION_MS = 3000;
  var stageEl = document.getElementById('stage');
  var boilHandle = null;
  var skipRequested = false;

  function resolveStage() {
    if (boilHandle) { boilHandle.stop(); boilHandle = null; }
    stageEl.classList.add('resolved');
    document.getElementById('boil-again').hidden =
      window.matchMedia('(prefers-reduced-motion: reduce)').matches;
  }

  function requestSkip() {
    skipRequested = true;
  }

  function runBoilSequence() {
    var reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    var alreadyBoiled = sessionStorage.getItem('boiled') === '1';

    if (reduce || alreadyBoiled) {
      resolveStage();
      return;
    }

    skipRequested = false;
    boilHandle = startBoilRender(document.getElementById('boil-canvas'));

    var skipListener = function() {
      requestSkip();
      cleanupSkipListeners();
    };
    function cleanupSkipListeners() {
      document.removeEventListener('click', skipListener);
      document.removeEventListener('keydown', skipListener);
      document.removeEventListener('touchstart', skipListener);
    }
    document.addEventListener('click', skipListener);
    document.addEventListener('keydown', skipListener);
    document.addEventListener('touchstart', skipListener);

    var start = performance.now();
    (function poll() {
      var elapsed = performance.now() - start;
      if (skipRequested || elapsed >= BOIL_DURATION_MS) {
        cleanupSkipListeners();
        sessionStorage.setItem('boiled', '1');
        resolveStage();
        return;
      }
      requestAnimationFrame(poll);
    })();
  }

  applyAmbientMotion();
  runBoilSequence();
</script>
```

- [ ] **Step 2: Manual Browser Verification — full boil**

Clear session storage first: DevTools → Application → Session Storage →
right-click the origin → Clear. Reload `http://localhost:8000/index.html`.

Expected: pot/burner/bubbles animate for ~3 seconds, then the canvas fades
out, the background warms to paper, the masthead fades in, and the four
bubbles scale/fade into place with their ambient bob already running. The
"Boil Again ↻" control fades in at the bottom.

- [ ] **Step 3: Manual Browser Verification — skip on interaction**

Clear session storage again, reload, and press any key (or click anywhere)
within the first second.

Expected: the resolved state appears immediately — no waiting out the rest
of the 3 seconds.

- [ ] **Step 4: Manual Browser Verification — session skip on return**

After either of the above completes, reload the page again (without
clearing session storage).

Expected: the page lands directly on the resolved state with no boil replay
at all — canvas never visibly renders the pot.

- [ ] **Step 5: Manual Browser Verification — reduced motion**

Clear session storage, enable "Emulate CSS prefers-reduced-motion: reduce"
in DevTools Rendering tab, reload.

Expected: page lands directly on resolved state, bubbles are static (no
bob), and the "Boil Again" control is hidden.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Orchestrate boil-to-resolve with session skip, click-to-skip, and reduced-motion"
```

---

### Task 6: "Boil Again" replay control

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `runBoilSequence()`, `resolveStage()`, `#stage` (Task 5).
- Produces: click handler on `#boil-again` that replays the boil without
  reloading the page and without disturbing the `sessionStorage` once-per-
  auto-visit rule.

- [ ] **Step 1: Add a manual-replay function and wire the button**

Add to the same script block from Task 5, replacing the final two lines
(`applyAmbientMotion(); runBoilSequence();`) with:

```html
<script>
  function replayBoilManually() {
    if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return;
    stageEl.classList.remove('resolved');
    skipRequested = false;
    boilHandle = startBoilRender(document.getElementById('boil-canvas'));

    var skipListener = function() { requestSkip(); cleanupSkipListeners(); };
    function cleanupSkipListeners() {
      document.removeEventListener('click', skipListener);
      document.removeEventListener('keydown', skipListener);
      document.removeEventListener('touchstart', skipListener);
    }
    // Give the click that triggered the replay a moment to not
    // immediately self-skip the animation it just started.
    setTimeout(function() {
      document.addEventListener('click', skipListener);
      document.addEventListener('keydown', skipListener);
      document.addEventListener('touchstart', skipListener);
    }, 300);

    var start = performance.now();
    (function poll() {
      var elapsed = performance.now() - start;
      if (skipRequested || elapsed >= BOIL_DURATION_MS) {
        cleanupSkipListeners();
        resolveStage();
        return;
      }
      requestAnimationFrame(poll);
    })();
  }

  document.getElementById('boil-again').addEventListener('click', replayBoilManually);

  applyAmbientMotion();
  runBoilSequence();
</script>
```

- [ ] **Step 2: Manual Browser Verification**

With the page in its resolved state, click "Boil Again ↻".

Expected: the stage darkens, pot/burner/bubbles replay for ~3 seconds
(without needing a page reload), then resolves again with bubbles back in
their resting positions and ambient bob restarted. Click "Boil Again" a
second time immediately after — it should be able to replay again (no
"only once ever" restriction). Reload the page afterward: it should land
straight on resolved state (confirming the manual replay never touched
`sessionStorage`).

- [ ] **Step 3: Manual Browser Verification — reduced motion**

With "prefers-reduced-motion: reduce" still emulated, confirm "Boil Again"
is not visible at all (per Task 5 Step 5's hiding logic), so there's nothing
to click.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add Boil Again control to manually replay the boil sequence"
```

---

### Task 7: Panel expand/collapse mechanics with focus management

**Files:**
- Modify: `index.html`

**Interfaces:**
- Produces: `openPanel(id)` / `closePanel()` functions; four
  `<div class="panel" id="panel-menu|chef|pass|foh">` overlay elements with
  placeholder headings (real content added in Tasks 8–10); click handlers
  on each `.bubble` calling `openPanel(this.dataset.panel)`; a `.panel-close`
  button and `Escape` key both call `closePanel()`.
- Consumes: `.bubble[data-panel]` buttons from Task 2.

- [ ] **Step 1: Add panel CSS**

Add to the `<style>` block:

```css
.panel{
  position:fixed;
  inset:0;
  background: var(--paper);
  overflow-y:auto;
  transform: scale(0.3);
  opacity:0;
  pointer-events:none;
  transition: transform 0.4s ease, opacity 0.4s ease;
  padding: 40px 24px 60px;
}
.panel.open{
  transform: scale(1);
  opacity:1;
  pointer-events:auto;
}
.panel[hidden]{ display:none; }
.panel-inner{ max-width: 680px; margin: 0 auto; }
.panel-close{
  position:sticky;
  top:0;
  float:right;
  font-family:'Space Mono', monospace;
  font-size: 22px;
  background: var(--paper);
  border: 2px solid var(--ember);
  color: var(--ember);
  width: 40px;
  height: 40px;
  border-radius: 50%;
  cursor:pointer;
  line-height:1;
}
@media (prefers-reduced-motion: reduce){
  .panel{ transition: none; }
}
```

- [ ] **Step 2: Add the four panel containers (placeholder content for now)**

Add right before the closing `</body>` tag:

```html
<div class="panel" id="panel-menu" role="dialog" aria-labelledby="panel-menu-title" hidden>
  <button class="panel-close" type="button" aria-label="Close">&times;</button>
  <div class="panel-inner">
    <h2 id="panel-menu-title">The Menu</h2>
    <p>Placeholder — real menu content added in Task 8.</p>
  </div>
</div>

<div class="panel" id="panel-chef" role="dialog" aria-labelledby="panel-chef-title" hidden>
  <button class="panel-close" type="button" aria-label="Close">&times;</button>
  <div class="panel-inner">
    <h2 id="panel-chef-title">The Chef</h2>
    <p>Placeholder — real content added in Task 9.</p>
  </div>
</div>

<div class="panel" id="panel-pass" role="dialog" aria-labelledby="panel-pass-title" hidden>
  <button class="panel-close" type="button" aria-label="Close">&times;</button>
  <div class="panel-inner">
    <h2 id="panel-pass-title">The Pass</h2>
    <p>Placeholder — real content added in Task 9.</p>
  </div>
</div>

<div class="panel" id="panel-foh" role="dialog" aria-labelledby="panel-foh-title" hidden>
  <button class="panel-close" type="button" aria-label="Close">&times;</button>
  <div class="panel-inner">
    <h2 id="panel-foh-title">Front of House</h2>
    <p>Placeholder — real content added in Task 10.</p>
  </div>
</div>
```

- [ ] **Step 3: Add the open/close logic and wire the bubble buttons**

Add a new script block after the boil-orchestration script:

```html
<script>
  var openPanelId = null;
  var lastFocusedTrigger = null;

  function onPanelKeydown(e) {
    if (e.key === 'Escape') closePanel();
  }

  function openPanel(id) {
    if (openPanelId) closePanel();
    var panel = document.getElementById('panel-' + id);
    if (!panel) return;
    lastFocusedTrigger = document.querySelector('.bubble[data-panel="' + id + '"]');
    panel.hidden = false;
    requestAnimationFrame(function() { panel.classList.add('open'); });
    openPanelId = id;
    panel.querySelector('.panel-close').focus();
    document.addEventListener('keydown', onPanelKeydown);
  }

  function closePanel() {
    if (!openPanelId) return;
    var panel = document.getElementById('panel-' + openPanelId);
    panel.classList.remove('open');
    var finish = function() { panel.hidden = true; };
    panel.addEventListener('transitionend', finish, { once: true });
    document.removeEventListener('keydown', onPanelKeydown);
    if (lastFocusedTrigger) lastFocusedTrigger.focus();
    openPanelId = null;
  }

  document.querySelectorAll('.bubble').forEach(function(bubble) {
    bubble.addEventListener('click', function() {
      openPanel(bubble.dataset.panel);
    });
  });

  document.querySelectorAll('.panel-close').forEach(function(btn) {
    btn.addEventListener('click', closePanel);
  });
</script>
```

- [ ] **Step 4: Manual Browser Verification**

Reload, let the boil resolve (or reload again to skip via session storage),
then click "The Menu" bubble.

Expected: the Menu panel expands to fill the viewport with focus landing on
its `×` button. Press `Esc`: panel shrinks away and focus returns to "The
Menu" bubble. Click "The Chef" bubble, then (while it's open) click "The
Pass" bubble: confirm only Pass ends up open (Chef closes automatically).
Click each panel's `×` and confirm each closes correctly. Tab through a
closed page: only the four bubbles and (if visible) Boil Again should be
reachable; when a panel is open, Tab should be able to reach its `×` and
back.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add panel expand/collapse mechanics with focus management"
```

---

### Task 8: The Menu panel — real content + shared form submit foundation

**Files:**
- Modify: `index.html`
- Read (for reference only, not modified): `menu.html`

**Interfaces:**
- Produces: `submitIntake(form, statusEl)` async function (shared by this
  panel's form and the Front of House form in Task 10); replaces the Menu
  panel placeholder with the real ticket content and a "printable menu →"
  link to `menu.html`.
- Consumes: `openPanel`/`closePanel` (Task 7).

- [ ] **Step 1: Replace the Menu panel placeholder with real content**

Replace the `#panel-menu` block from Task 7 with:

```html
<div class="panel" id="panel-menu" role="dialog" aria-labelledby="panel-menu-title" hidden>
  <button class="panel-close" type="button" aria-label="Close">&times;</button>
  <div class="panel-inner">
    <div class="eyebrow">Today's Menu <span class="note">— build price includes first year of Web Care</span></div>
    <h2 id="panel-menu-title" style="font-family:'Space Mono',monospace; margin-bottom:16px;">The Menu</h2>

    <div class="course-label">Builds</div>
    <p class="scale-intro">Builds are priced on a sliding scale — pay what you can afford within the range shown for each one.</p>

    <div class="menu-item scale-row">
      <div class="desc">
        <div class="name">Artist / Brand Website</div>
        <div class="sub">Custom-built, responsive · Year 1 Care included</div>
      </div>
      <div class="scale-price">
        <span class="scale-stamp">Sliding Scale</span>
        <div class="price"><span class="tag">Suggested</span>$420</div>
        <div class="scale-range"><span class="bracket">[</span>$200 – $1,000<span class="bracket">]</span></div>
      </div>
    </div>

    <div class="menu-item scale-row">
      <div class="desc">
        <div class="name">Web Application</div>
        <div class="sub">React/Node front-to-back, database included · Year 1 Care included</div>
      </div>
      <div class="scale-price">
        <span class="scale-stamp">Sliding Scale</span>
        <div class="price"><span class="tag">Suggested</span>$1,200</div>
        <div class="scale-range"><span class="bracket">[</span>$500 – $2,000<span class="bracket">]</span></div>
      </div>
    </div>

    <div class="menu-item scale-row">
      <div class="desc">
        <div class="name">E-commerce Build</div>
        <div class="sub">Storefront, cart, payments · Years 1–2 Care included</div>
      </div>
      <div class="scale-price">
        <span class="scale-stamp">Sliding Scale</span>
        <div class="price"><span class="tag">Suggested</span>$2,250</div>
        <div class="scale-range"><span class="bracket">[</span>$1,500 – $3,000<span class="bracket">]</span></div>
      </div>
    </div>

    <div class="course-label">Web Care — after your included period</div>

    <div class="menu-item">
      <div class="desc"><div class="name">Light Care</div><div class="sub">A few updates a year, domain/email upkeep</div></div>
      <div class="leader"></div>
      <div class="price">$45<span class="per">/mo</span></div>
    </div>
    <div class="menu-item">
      <div class="desc"><div class="name">Active Care</div><div class="sub">Monthly updates, inbox forwarding, priority turnaround</div></div>
      <div class="leader"></div>
      <div class="price">$125<span class="per">/mo</span></div>
    </div>
    <div class="menu-item">
      <div class="desc"><div class="name">Org Care</div><div class="sub">For businesses, not solo creators — heavier hours, faster SLA</div></div>
      <div class="leader"></div>
      <div class="price">$425<span class="per">/mo</span></div>
    </div>
    <div class="menu-item">
      <div class="desc"><div class="name">À la Carte Fix</div><div class="sub">One-off, no subscription. Don't panic.</div></div>
      <div class="leader"></div>
      <div class="price">$42<span class="per">/hr</span></div>
    </div>
    <div class="menu-item">
      <div class="desc"><div class="name">Custom Project</div><div class="sub">Tell me what you're building</div></div>
      <div class="leader"></div>
      <div class="price">quote on request</div>
    </div>

    <div class="care-note">
      <b>How Care tiers work:</b> pricing scales with how often you need touched — someone who
      emails changes monthly pays a different rate than someone who checks in every couple of
      years. Tier gets set at your intake based on how you'll actually use the site.
    </div>

    <form class="intake" id="menuIntakeForm" style="margin-top:24px;">
      <div class="field"><label for="m-name">Your Name</label><input type="text" id="m-name" name="name" required></div>
      <div class="field"><label for="m-email">Email</label><input type="email" id="m-email" name="email" required></div>
      <div class="field"><label for="m-what">What's the site/app for?</label><textarea id="m-what" name="project_description" placeholder="e.g. portfolio for my paintings, storefront for handmade jewelry"></textarea></div>
      <button type="submit">Send Order to the Kitchen</button>
      <p class="form-status" id="menuFormStatus" hidden></p>
      <p style="font-size:12px; margin-top:10px; color:var(--slate);">or just email <a href="mailto:solutions@chefmyklove.com">solutions@chefmyklove.com</a> directly</p>
    </form>

    <p style="margin-top:20px;"><a href="menu.html" target="_blank" rel="noopener">printable menu &rarr;</a></p>
  </div>
</div>
```

- [ ] **Step 2: Copy the `.eyebrow`, `.course-label`, `.menu-item`, `.scale-*`, `.care-note`, `.intake` CSS rules from `menu.html` into `index.html`'s `<style>` block**

These class names and their rules already exist verbatim in `menu.html`
(lines defining `.eyebrow`, `.course-label`, `.menu-item`, `.leader`,
`.price`, `.scale-intro`, `.scale-stamp`, `.scale-range`, `.care-note`,
`.intake` and its children). Copy that whole set of rules as-is into the
new `index.html`'s `<style>` block so the Menu panel renders identically.

- [ ] **Step 3: Add the shared `submitIntake` function**

Add a new script block:

```html
<script>
  var WEB3FORMS_ACCESS_KEY = 'REPLACE_WITH_YOUR_WEB3FORMS_ACCESS_KEY';

  async function submitIntake(form, statusEl) {
    var formData = new FormData(form);
    formData.append('access_key', WEB3FORMS_ACCESS_KEY);
    statusEl.hidden = false;
    statusEl.textContent = 'Sending...';
    try {
      var res = await fetch('https://api.web3forms.com/submit', {
        method: 'POST',
        body: formData
      });
      var data = await res.json();
      if (data.success) {
        statusEl.textContent = "Order received — I'll be in touch soon.";
        form.reset();
      } else {
        throw new Error(data.message || 'Submission failed');
      }
    } catch (err) {
      statusEl.textContent =
        "Something didn't fire — email solutions@chefmyklove.com directly and I'll get it.";
    }
  }

  document.getElementById('menuIntakeForm').addEventListener('submit', function(e) {
    e.preventDefault();
    submitIntake(this, document.getElementById('menuFormStatus'));
  });
</script>
```

- [ ] **Step 4: Manual Browser Verification**

Open the Menu panel. Expected: pricing content renders with correct
typography/spacing matching `menu.html`'s look. Fill in the form and submit
with `WEB3FORMS_ACCESS_KEY` still at its placeholder value — expected the
`fetch` to fail or return an error, and the status text to show the
"Something didn't fire — email solutions@chefmyklove.com directly" fallback
message (this confirms the failure path works; Task 10 covers verifying an
actual successful submission once a real key is supplied). Click "printable
menu →" and confirm it opens `menu.html` in a new tab correctly.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add real Menu panel content and shared Web3Forms submit handler"
```

---

### Task 9: The Chef and The Pass panels

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `openPanel`/`closePanel` (Task 7).
- Produces: final copy for `#panel-chef` and `#panel-pass`, with
  `[BRACKETED]` placeholders per spec §7.2/§7.3 for the author to fill in.

- [ ] **Step 1: Replace the Chef panel placeholder**

```html
<div class="panel" id="panel-chef" role="dialog" aria-labelledby="panel-chef-title" hidden>
  <button class="panel-close" type="button" aria-label="Close">&times;</button>
  <div class="panel-inner">
    <div class="eyebrow">The Chef</div>
    <h2 id="panel-chef-title" style="font-family:'Space Mono',monospace; margin-bottom:16px;">Confidential</h2>
    <p style="margin-bottom:14px; line-height:1.7;">
      I worked kitchens for <strong>[X YEARS]</strong>, mostly as
      <strong>[YOUR ROLE(S) — e.g. line cook / sous chef]</strong> at
      <strong>[KIND OF PLACES — e.g. a couple of mid-size scratch kitchens and one place that
      should've closed a year before it did]</strong>. I left because
      <strong>[YOUR REAL REASON]</strong>, and landed in software around
      <strong>[YEAR / HOW IT HAPPENED]</strong>.
    </p>
    <p style="margin-bottom:14px; line-height:1.7;">
      Here's the thing nobody warns you about: the two jobs run on the same discipline. Mise en
      place is just sprint planning with knives. "The ticket's not done till the plate's right"
      is the same standard as "the feature's not done till it works in prod." And the reason my
      builds all ship with a year of Web Care isn't a sales tactic — it's the same instinct that
      keeps a line cook from walking out mid-service. You don't hand someone a plate and
      disappear. You don't hand someone a website and disappear either.
    </p>
    <p style="line-height:1.7;">
      <strong>[OPTIONAL: ONE MORE SPECIFIC KITCHEN DETAIL/STORY THAT MAKES THIS REAL]</strong>
    </p>
  </div>
</div>
```

- [ ] **Step 2: Replace the Pass panel placeholder**

```html
<div class="panel" id="panel-pass" role="dialog" aria-labelledby="panel-pass-title" hidden>
  <button class="panel-close" type="button" aria-label="Close">&times;</button>
  <div class="panel-inner">
    <div class="eyebrow">The Pass — Recent Work</div>
    <h2 id="panel-pass-title" style="font-family:'Space Mono',monospace; margin-bottom:16px;">Tickets Fired</h2>

    <div class="tasting-item">
      <div class="top">
        <span class="site"><a href="https://kathleenyearwood.com" target="_blank" rel="noopener">kathleenyearwood.com</a></span>
        <span class="tag">Artist website</span>
      </div>
      <div class="note">
        Stack: <strong>[e.g. static HTML/CSS, or React — fill in]</strong>. Constraint:
        <strong>[REAL CONSTRAINT — budget, timeline, client's existing brand assets, etc.]</strong>.
        One decision worth telling: <strong>[e.g. why you structured the gallery / nav / hosting
        the way you did]</strong>.
      </div>
    </div>

    <div class="tasting-item">
      <div class="top">
        <span class="site"><a href="https://chefmyklove.github.io/SelinaMartin/" target="_blank" rel="noopener">selinamartin.com</a></span>
        <span class="tag">Artist website</span>
      </div>
      <div class="note">
        Stack: <strong>[FILL IN]</strong>. Constraint: <strong>[FILL IN]</strong>. Decision:
        <strong>[FILL IN]</strong>.
      </div>
    </div>

    <div class="tasting-item">
      <div class="top">
        <span class="site"><a href="https://chefmyklove.com" target="_blank" rel="noopener">chefmyklove.com</a></span>
        <span class="tag">Studio site</span>
      </div>
      <div class="note">
        Stack: <strong>[FILL IN]</strong>. Constraint: <strong>[FILL IN]</strong>. Decision:
        <strong>[FILL IN]</strong>.
      </div>
    </div>

    <div class="tasting-item" style="border-bottom:none;">
      <div class="top"><span class="site">This page, too</span><span class="tag">Exhibit</span></div>
      <div class="note">
        The boil above is hand-rolled Canvas 2D — no animation library, no framework, zero
        dependencies. If you want to see how it works, view source. That's not a dare, that's
        the pitch.
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 3: Copy the `.tasting-item` CSS rules from `menu.html` into `index.html`'s `<style>` block**

(`.tasting-item`, `.tasting-item .top`, `.tasting-item .site`,
`.tasting-item a`, `.tasting-item .note`, `.tasting-item .tag` — copy
verbatim from `menu.html`.)

- [ ] **Step 4: Manual Browser Verification**

Open The Chef panel: confirm the bracketed placeholders are clearly visible
(not silently dropped) and the copy reads coherently around them. Open The
Pass panel: confirm all three site links open correctly in new tabs and the
"This page, too" exhibit block renders as the last item with no bottom
border artifact.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add Chef and Pass panel content with flagged fact placeholders"
```

---

### Task 10: Front of House panel — contact form + Web3Forms wiring

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `submitIntake(form, statusEl)` (Task 8).
- Produces: real Front of House panel content, reusing the same intake
  field set as the current ticket (name, email, project description,
  project type, update frequency, managed-email need).

- [ ] **Step 1: Replace the Front of House panel placeholder**

```html
<div class="panel" id="panel-foh" role="dialog" aria-labelledby="panel-foh-title" hidden>
  <button class="panel-close" type="button" aria-label="Close">&times;</button>
  <div class="panel-inner">
    <div class="eyebrow">Front of House — Start an Order</div>
    <h2 id="panel-foh-title" style="font-family:'Space Mono',monospace; margin-bottom:16px;">Tell Me About Your Project</h2>

    <div class="contact-grid" style="margin-bottom:24px;">
      <div><div class="label">Email</div><div class="value"><a href="mailto:solutions@chefmyklove.com">solutions@chefmyklove.com</a></div></div>
      <div><div class="label">Messenger</div><div class="value">Chefmyklove Custom Software Solutions</div></div>
      <div><div class="label">WhatsApp</div><div class="value">Available on request</div></div>
      <div><div class="label">Web</div><div class="value"><a href="https://solutions.chefmyklove.com" target="_blank" rel="noopener">solutions.chefmyklove.com</a></div></div>
    </div>

    <form class="intake" id="fohIntakeForm">
      <div class="field"><label for="f-name">Your Name</label><input type="text" id="f-name" name="name" required></div>
      <div class="field"><label for="f-email">Email</label><input type="email" id="f-email" name="email" required></div>
      <div class="field"><label for="f-what">What's the site/app for?</label><textarea id="f-what" name="project_description" placeholder="e.g. portfolio for my paintings, storefront for handmade jewelry, booking tool for my studio"></textarea></div>
      <div class="field">
        <label for="f-type">Closest to</label>
        <select id="f-type" name="project_type">
          <option>Artist / Brand Website</option>
          <option>Web Application</option>
          <option>E-commerce Build</option>
          <option>Not sure yet</option>
        </select>
      </div>
      <div class="field">
        <label>Do you need it maintained/updated regularly?</label>
        <div class="checks">
          <label><input type="radio" name="freq" value="Rarely (every year or two)" checked> Rarely</label>
          <label><input type="radio" name="freq" value="A few times a year"> A few times a year</label>
          <label><input type="radio" name="freq" value="Monthly or more"> Monthly+</label>
        </div>
      </div>
      <div class="field">
        <label>Need a managed email address too?</label>
        <div class="checks">
          <label><input type="radio" name="email_need" value="Yes" checked> Yes</label>
          <label><input type="radio" name="email_need" value="No"> No</label>
        </div>
      </div>
      <button type="submit">Send Order to the Kitchen</button>
      <p class="form-status" id="fohFormStatus" hidden></p>
      <p style="font-size:12px; margin-top:10px; color:var(--slate);">or just email <a href="mailto:solutions@chefmyklove.com">solutions@chefmyklove.com</a> directly</p>
    </form>
  </div>
</div>
```

- [ ] **Step 2: Copy the `.contact-grid` CSS rules from `menu.html`, and add a `.form-status` rule**

Copy `.contact-grid`, `.contact-grid .label`, `.contact-grid .value`,
`.contact-grid .value a` verbatim from `menu.html`. Add:

```css
.form-status{
  font-size: 13px;
  margin-top: 10px;
  color: var(--sage);
}
```

- [ ] **Step 3: Wire the form's submit handler**

Add to the script block that already wires `menuIntakeForm` (Task 8, Step
3):

```html
<script>
  document.getElementById('fohIntakeForm').addEventListener('submit', function(e) {
    e.preventDefault();
    submitIntake(this, document.getElementById('fohFormStatus'));
  });
</script>
```

(Add this as its own small script block right after the existing one, or
append the `addEventListener` call directly into the existing block — both
work since `submitIntake` is already in scope.)

- [ ] **Step 4: Manual Browser Verification — full real submission**

Create a free Web3Forms account at web3forms.com under
`solutions@chefmyklove.com`, get a real access key, and temporarily paste
it into `WEB3FORMS_ACCESS_KEY` locally (do not commit the real key yet —
see Step 6). Reload, open Front of House, fill out the form with real test
data, and submit.

Expected: status text changes to "Sending..." then to "Order received —
I'll be in touch soon.", the form fields clear, and an email arrives at
`solutions@chefmyklove.com` (check inbox) containing the submitted fields.
Repeat the same real-key test on the Menu panel's form from Task 8.

- [ ] **Step 5: Manual Browser Verification — failure fallback still visible**

Temporarily disable network access (DevTools → Network → Offline), submit
either form again.

Expected: status text shows the "Something didn't fire — email
solutions@chefmyklove.com directly" fallback, and the always-visible "or
just email..." line beneath the button was visible the entire time,
independent of submission state.

- [ ] **Step 6: Revert the access key placeholder before committing**

```bash
git diff index.html | grep WEB3FORMS_ACCESS_KEY
```

Confirm the committed value is the placeholder string
`REPLACE_WITH_YOUR_WEB3FORMS_ACCESS_KEY`, not a real key (the author swaps
in the real key locally post-deploy, per spec §10 open item #4 — this key
is a secret and should not live in git history).

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "Add Front of House panel with contact form and Web3Forms wiring"
```

---

### Task 11: Mobile layout pass, meta polish, and final QA checklist

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: everything from Tasks 1–10.
- Produces: no new functions — this task is verification, tuning, and
  small CSS fixes discovered during the checklist below.

- [ ] **Step 1: Verify and fix mobile panel layout**

At a viewport narrower than 700px (DevTools device toolbar, e.g. iPhone SE
375px), open each of the four panels.

Expected: panel content reflows to single-column, no horizontal scrollbar,
the `×` close button remains reachable and tappable (not obscured by
notches/status bars). If any panel overflows horizontally (likely the
`.menu-item.scale-row` or `.contact-grid`), reuse the existing responsive
rules already defined for those classes in `menu.html`'s `@media
(max-width: 480px)` block — copy that block's relevant rules
(`.contact-grid{ grid-template-columns: 1fr; }`,
`.menu-item.scale-row{ flex-direction:column; ... }`, etc.) into
`index.html`'s `<style>` block if not already present from earlier tasks.

- [ ] **Step 2: Verify ambient bubble count on mobile**

At the same narrow viewport, clear session storage and reload to watch the
boil.

Expected: `startBoilRender`'s `maxBubbles` value (14 on screens under
700px, set in Task 4 Step 4) keeps the animation smooth — no visible
stutter. If DevTools Performance shows dropped frames, lower the `14` in
`var maxBubbles = window.innerWidth < 700 ? 14 : 26;` further (e.g. to `10`)
and re-check.

- [ ] **Step 3: Run the full manual QA checklist from the spec**

Walk through, on a real desktop browser (not just DevTools emulation where
possible):

1. Boil → resolved transition plays once, looks intentional, no console errors.
2. Click/keypress during boil skips immediately.
3. Reload after resolving lands directly on resolved state (no replay).
4. `prefers-reduced-motion: reduce` fully disables boil + ambient drift; Boil Again is hidden.
5. "Boil Again" replays on demand and does not affect the session-skip rule on subsequent reloads.
6. Keyboard-only pass: Tab reaches all four bubbles (and Boil Again when visible) in a sane order; Enter/Space opens a panel; focus moves to the panel's close button; Esc closes and returns focus to the originating bubble.
7. Mobile 2×2 layout at <700px confirmed in Step 1/2 above.
8. Real form submission through Web3Forms succeeds end-to-end (Task 10, Step 4) and the error-fallback path displays correctly offline (Task 10, Step 5).
9. `menu.html` still renders and prints correctly standalone (re-check Task 1, Step 3 — nothing since Task 1 should have touched it).

- [ ] **Step 4: Confirm all remaining `[BRACKETED]` placeholders are visible and untouched**

```bash
grep -n "\[.*\]" index.html
```

Expected: only the intentional copy placeholders from Task 9 appear (Chef
bio facts, Pass case-study facts) plus `WEB3FORMS_ACCESS_KEY`'s placeholder
string — nothing else. This is the author's remaining to-do list per spec
§10, not a defect in this plan.

- [ ] **Step 5: Final commit**

```bash
git add index.html menu.html
git commit -m "Mobile layout fixes and final QA pass for stockpot splash"
```

---

## Self-Review Notes

- **Spec coverage:** §2 (concept/animation) → Tasks 2–6. §3 (files) → Tasks
  1–2. §4 (visual system) → Tasks 2, 8, 9, 10 (shared palette/typography
  reused throughout, no competing palette introduced). §5 (animation
  rules incl. Boil Again) → Tasks 3, 4, 5, 6. §6 (panel behavior) → Task 7.
  §7.1–7.4 (panel contents) → Tasks 8, 9, 10. §8 (error handling/testing) →
  Task 10 Steps 4–5, Task 11 Step 3. §9 (out of scope) → nothing built for
  GitHub links, routing, frameworks, analytics, or literal cover pastiche,
  confirmed absent throughout. §10 (open author items) → flagged explicitly
  in Tasks 9 and 10, and swept for completeness in Task 11 Step 4.
- **Placeholder scan:** the only bracketed/placeholder strings intentionally
  left in the shipped code are the author-facing content fill-ins (spec
  §10) and the Web3Forms key — every other step has concrete, runnable
  code.
- **Type/name consistency:** `openPanel`/`closePanel` (Task 7) are reused
  unchanged by Tasks 8–10; `submitIntake(form, statusEl)` (Task 8) is reused
  with the same signature by Task 10; `startBoilRender(canvas)` (Task 4) is
  reused unchanged by Tasks 5 and 6; `resolveStage()` and `requestSkip()`
  (Task 5) are reused by Task 6's manual replay path.

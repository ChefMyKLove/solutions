# The Stockpot — Splash + Bubble Navigation

Date: 2026-07-21
Status: Approved, ready for implementation planning

## 1. Purpose

Replace the current site root (a services/pricing ticket) with a compelling,
animated splash page that establishes a distinct visual identity, then
resolves into a modular bubble-based navigation. Goal: generate freelance
leads and catch the attention of employers evaluating the author as a
software craftsperson. The existing ticket becomes one destination (Menu)
among several, not the front door.

Design constraint carried through every decision below: **no AI slop**. No
stock-template animation libraries, no generic "hero + fade-in" patterns, no
invented biographical claims. Everything hand-built, everything true.

## 2. Concept

The site opens dark. A burner clicks twice and catches — an ember-red ring
glows under a black stockpot silhouette rendered on canvas. Small bubbles
form at the bottom of the pot, wobble upward, and pop at the surface. The
boil builds for roughly 3 seconds. Then four bubbles, instead of popping,
grow larger, break the surface, and rise as the pot and burner fade out. The
background warms from charcoal to the site's existing paper cream as the
four bubbles settle into a loose, gentle drift near the center of the
viewport:

- **The Menu** — services & pricing (existing ticket content)
- **The Chef** — about / the kitchen-to-code story
- **The Pass** — examples & case studies
- **Front of House** — contact form

Bubbles bob independently, softly collide and separate, and wobble on
hover/focus. Above them: the author's name and a tagline in brash
condensed red/black display type on cream — a Kitchen Confidential
first-edition homage expressed through **palette, type attitude, and
confessional first-person voice**, not literal cover pastiche.

Working tagline (author to approve or replace before ship):
> "Custom software from a guy who used to work the line."

Clicking a bubble expands it in place to fill the viewport as a content
panel. A close control (`×` button, and `Esc` key) shrinks the panel back
into the resolved boil state. The whole experience is a single page — no
reloads, and the boil animation never re-runs while navigating between
panels.

## 3. Structure & Files

- **`index.html`** — new splash entry point. All CSS, vanilla JS, and the
  Canvas 2D bubble simulation are inline in this one file, matching the
  existing site's single-file convention. **Zero external JS
  dependencies** — no libraries, no build tooling, no bundler. Code is
  commented for a human reader, since view-source is treated as part of
  the portfolio pitch, not an afterthought.
- **`menu.html`** — the current `index.html` content, moved here verbatim
  (ticket layout, print styles, QR code, intake form). Serves as the
  printable / directly-linkable version of the Menu panel. The Menu panel
  inside the splash renders the same content and links out to
  `menu.html` via a "printable menu →" footer link.
- No new build system, framework, or package.json. This stays a static
  site of hand-authored HTML/CSS/JS files, matching the current repo.

## 4. Visual System

- Reuses the existing palette variables from the current ticket design:
  `--paper`, `--paper-shadow`, `--char`, `--ember`, `--sage`, `--slate`,
  `--line`. The splash pushes the red/black contrast harder in the
  pre-boil dark state and the headline treatment, but never introduces a
  competing palette — so the Menu panel (built from the existing ticket
  styles) doesn't clash when it expands.
- One additional display font for the headline/tagline: a condensed
  grotesque (e.g. Archivo Black or similar), loaded the same way the
  existing Google Fonts are loaded (`Space Mono` / `Work Sans` stay for
  body/mono use inside panels). Final font pick happens during
  implementation and is a low-risk, easily swapped detail.
- Bubble buttons are visually simple: circular, paper-toned with ember
  outline/label, sized to comfortably contain a short label (MENU / THE
  CHEF / THE PASS / CONTACT or similar short forms — exact bubble labels
  finalized during implementation, kept short for legibility at bubble
  scale).

## 5. Animation Behavior — the no-slop / accessibility rules

- The boil sequence plays **once per browser session**
  (`sessionStorage` flag). Returning to the page in the same session (or
  navigating back from a panel) lands directly on the resolved,
  drifting-bubbles state — it does not replay the boil.
- **Any click, tap, or keypress during the boil skips immediately** to the
  resolved bubble state. The animation is never a mandatory gate.
- **`prefers-reduced-motion: reduce`**: the boil sequence and the ambient
  drift are both skipped/disabled entirely. Bubbles render statically in
  place, fully functional, with hover/focus states still working (those
  are not considered "motion" in the disruptive sense, but any
  transition is kept minimal/instant under this setting).
- Bubble navigation buttons are **real `<button>` elements in the DOM**,
  positioned absolutely to line up with their canvas-rendered bubble
  visuals. The canvas layer is `aria-hidden="true"` and purely
  decorative/presentational. This means keyboard navigation (Tab, Enter,
  Space) and screen readers interact with ordinary buttons and never
  depend on canvas content — full functionality with zero canvas
  involvement.
- Mobile: same sequence and rules, with a reduced particle count for
  performance, and the four resolved bubbles arranged in a 2×2 grid
  instead of a free scatter.

## 6. Panel Behavior

- Clicking a bubble button triggers its bubble (and the DOM panel behind
  it) to scale/expand to fill the viewport, revealing scrollable content
  inside.
- A `×` close control (top corner of the panel) and the `Esc` key both
  collapse the panel back to the resolved bubble-drift state — the same
  state the boil resolves into, not a re-run of the boil.
- Only one panel is open at a time. Opening a new bubble while one is
  open closes the previous one.
- No routing, no hash URLs, no browser history entries for panel
  open/close (confirmed out of scope — see §9).

## 7. Panel Contents

### 7.1 The Menu
The existing ticket content (Today's Menu: Builds, Web Care tiers,
sliding-scale pricing) reused as-is inside the panel. The intake form here
receives the same submission upgrade as the Contact panel (§7.4). Footer
of this panel links to `menu.html` labeled "printable menu →".

### 7.2 The Chef
First-person confessional copy in the Kitchen-Confidential-influenced
voice, drafted with `[BRACKETED]` placeholders for every factual claim —
years on the line, specific roles (line cook / sous / other), the kind of
kitchens worked, why the author left food service, and when/how software
entered the picture. **No fact is invented.** The narrative throughline
connects kitchen discipline to engineering discipline: mise en place as
prep/planning, "done means done" as a delivery standard, nobody walking
out mid-service as the argument underpinning the existing Web Care
offering (ongoing care is table stakes, not an upsell). All bracketed
placeholders must be filled in by the author before this panel ships;
implementation will flag every placeholder clearly.

### 7.3 The Pass
The three existing live client sites (kathleenyearwood.com,
selinamartin.com, chefmyklove.com), each with a short case-study block:
stack used, a real constraint, and one build/design decision worth
telling — placeholdered with `[BRACKETED]` slots wherever the true
specifics aren't yet known, to be filled in before ship (same rule as
§7.2: no invented detail). GitHub links are explicitly excluded per
author's direction. Closing exhibit: two sentences pointing at the splash
page itself as a work sample — noting it's hand-rolled Canvas 2D with zero
dependencies — with an invitation to view source.

### 7.4 Front of House
The existing intake form fields (name, email, project description,
project type, update frequency, managed-email need), submitting via a
**Web3Forms** POST (free static-site form endpoint) that delivers
submissions as email to `solutions@chefmyklove.com`. A visible fallback
line — "or just email solutions@chefmyklove.com directly" — sits below the
submit button at all times, not only on failure. On a failed/errored
submission, an inline message appears reiterating the direct-email
fallback. Requires a Web3Forms access key (author creates a free account
under their email; the key is a single placeholder constant in the code,
clearly flagged for the author to fill in before the form goes live).

## 8. Error Handling & Testing

- Form submission failure (network error or non-2xx response from
  Web3Forms): show an inline error message plus the direct-email
  fallback link — never a silent failure.
- Before this is considered done, verify in a real desktop browser:
  the boil → resolved-bubbles transition, the skip-on-interaction path,
  `prefers-reduced-motion` behavior, keyboard-only navigation through all
  four bubbles and panel close (Esc and ×), the mobile 2×2 layout at a
  narrow viewport, a real form submission through Web3Forms, and that
  `menu.html` still renders/prints correctly on its own.

## 9. Explicitly Out of Scope

- GitHub/code links in The Pass (author's call).
- Separate per-section URLs, hash-based deep links, or browser history
  entries for panel navigation — this is a single-page expand/collapse
  experience only.
- Any JavaScript framework, bundler, or build step.
- Analytics/tracking of any kind.
- Literal Kitchen Confidential cover pastiche — the homage is limited to
  palette, type attitude, and voice.

## 10. Open Items for the Author (must be resolved before ship)

1. Fill in all `[BRACKETED]` factual placeholders in The Chef panel copy.
2. Fill in all `[BRACKETED]` factual placeholders in The Pass case-study
   blocks.
3. Approve or replace the working tagline.
4. Create a Web3Forms account/access key and supply it for the Contact
   and Menu-panel forms.
5. Approve final bubble label wording (short forms of Menu / The Chef /
   The Pass / Front of House) once seen at actual bubble scale.

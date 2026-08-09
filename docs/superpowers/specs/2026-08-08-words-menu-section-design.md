# Words Menu Section — Grantwriting & Copywriting

## Context

The site owner is adding grantwriting and copywriting to the services offered on
solutions.chefmyklove.com. The site's "Today's Menu" panel (`#panel-menu` in
`index.html`, mirrored in the printable `menu.html`) currently has two course
sections:

- **Builds** — one-time website projects, sliding-scale pricing (suggested price
  + a `[$low – $high]` range), using `.menu-item.scale-row`
- **Web Care** — recurring maintenance retainers, flat pricing, using plain
  `.menu-item` rows

This adds a third course, **Words**, for the new writing services, using the
same visual language already established by these two sections.

## Design

### Placement

`Words` is inserted as a new `.course-label` section directly after `Builds`
and before `Web Care` — both `Builds` and `Words` are one-time,
sliding-scale-priced project work; `Web Care` is the recurring/subscription
section. Grouping by pricing model (one-time vs. recurring) keeps the existing
menu logic intact rather than introducing a third grouping principle.

No `scale-intro` line is needed above `Words` — the existing one above
`Builds` ("Builds are priced on a sliding scale...") already explains the
sliding-scale convention, and `Words`'s two scale-rows sit close enough below
it to read as covered by the same explanation.

### Content

Three rows, in this order:

1. **Grant Application Writing** — sub: "Research + full application, one
   grant" — `.scale-row` — stamp "Sliding Scale" — suggested `$420` — range
   `$250 – $750`
2. **Website / Marketing Copy** — sub: "Site pages, product/brand copy" —
   `.scale-row` — stamp "Sliding Scale" — suggested `$420` — range
   `$250 – $750`
3. **Content & Email Package** — sub: "Blog posts, newsletters, email
   sequences — bundled as a starter set" — plain `.menu-item` (not
   scale-row) — price text "quote on request", matching the existing
   "Custom Project" row under Web Care

Markup follows the exact structure already used for the Builds rows (rows 1–2)
and the Web Care "Custom Project" row (row 3) — no new CSS classes needed,
this is pure content addition using existing, already-styled patterns.

### Intake form

The existing single intake form (`#menuIntakeForm`) below all three courses
already serves every menu item — no new form or new fields. The placeholder
text on the "What's the site/app for?" textarea is loosened slightly (e.g. to
"e.g. portfolio for my paintings, storefront for handmade jewelry, or a grant
application / copywriting project") so writing-service inquiries don't feel
like they're being forced into a website-shaped question.

### Two files, kept in sync

`menu.html` is a standalone printable page that duplicates the same
`course-label` / `menu-item` markup found in `index.html`'s `#panel-menu`.
Both files get the identical `Words` section added in the identical position,
so the printable menu and the live site panel stay in sync. No shared
template/include exists between them today, so this is a manual duplication,
consistent with how `Builds` and `Web Care` are already duplicated between the
two files.

### Out of scope

- No new nav bubble, panel, or page — `Words` lives inside the existing menu
  panel per the site owner's decision.
- No changes to the `Web Care` section, the `care-note`, or the eyebrow note
  above the menu title (it specifically references Builds' included Web Care
  year and doesn't apply to Words).
- No changes to pricing logic/scripting — this is static content, matching
  the rest of the menu panel.

---
name: clone-website
description: "Reverse-engineer and clone one or more websites as pixel-perfect replicas"
invokable: true
---
<!-- AUTO-GENERATED from .claude/skills/clone-website/SKILL.md — do not edit directly.
     Run `node scripts/sync-skills.mjs` to regenerate. -->


# Clone Website

You are about to reverse-engineer and rebuild **$ARGUMENTS** as pixel-perfect clones.

When multiple URLs are provided, preserve every pathname as a distinct route and isolate each target's research, screenshots, components, and assets. URLs that differ only by query string or fragment share a pathname, so resolve their route and state behavior explicitly in the output plan. Parallelize page work only after the shared foundation and output plan are fixed so concurrent builders cannot overwrite one another.

This is not a two-phase process (inspect then build). You are a **foreman walking the job site** — as you inspect each section of the page, you write a detailed specification to a file, then hand that file to a specialist builder agent with everything they need. Extraction and construction happen in parallel, but extraction is meticulous and produces auditable artifacts.

## Scope Defaults

The target is whatever page `$ARGUMENTS` resolves to. Clone exactly what's visible at that URL. Unless the user specifies otherwise, use these defaults:

- **Fidelity level:** Pixel-perfect — exact match in colors, spacing, typography, animations
- **In scope:** Visual layout and styling, component structure and interactions, responsive design, mock data for demo purposes
- **Out of scope:** Real backend / database, authentication, real-time features, SEO optimization, accessibility audit
- **Customization:** None — pure emulation

If the user provides additional instructions (specific fidelity level, customizations, extra context), honor those over the defaults.

## Output Isolation and Route Preservation

Treat every target URL as durable project output, not as permission to replace whatever was built previously.

Choose an `<app-root>` before extraction. For a single application, `<app-root>` is the repository root (`.`). If different origins need separate applications, require the user to provide or approve a prepared Next.js project root for each origin; verify each root builds independently, and never write one origin's output into another root.

Then assign each target:

- A collision-resistant `<site-key>`: a readable origin slug (including a non-default port) plus the first 8 lowercase hex characters of SHA-256 over the normalized origin.
- A collision-resistant `<page-key>`: a segment-preserving readable pathname slug plus the first 8 lowercase hex characters of SHA-256 over the normalized pathname and any stateful query/fragment; use `root-<hash>` for `/`. Never rely on lossy character replacement alone.
- An artifact root: `<app-root>/docs/research/<site-key>/<page-key>/`.
- A screenshot root: `<app-root>/docs/design-references/<site-key>/<page-key>/`.
- A component root: `<app-root>/src/components/sites/<site-key>/<page-key>/`, with genuinely shared same-site components under `<app-root>/src/components/sites/<site-key>/shared/`.
- An asset root: `<app-root>/public/sites/<site-key>/<page-key>/`, with genuinely shared same-site assets under `<app-root>/public/sites/<site-key>/shared/`.
- A Next.js route file.

All paths in the remaining phases are relative to that target's `<app-root>`. Before writing, verify that every planned route, artifact root, screenshot root, component root, asset root, and downloader filename is unique or is an explicitly approved shared location.

Routing defaults:

- For the first single-URL clone in an untouched template, the existing scaffold at `src/app/page.tsx` may be replaced so the clone remains available at `/`.
- For multiple URLs from the same origin, or any later clone added to a project that already contains cloned/user-authored pages, preserve the normalized source pathname as its App Router URL (for example, `/docs/intro` becomes `<app-root>/src/app/docs/intro/page.tsx`). Encode filesystem segment names that would invoke App Router syntax: escape a leading `_` or `@`, and literal parentheses or square brackets, with percent-encoded folder spellings rather than creating private folders, slots, route groups, or dynamic segments. Verify the built route resolves at the exact normalized URL before completion.
- Inspect every existing `src/app/**/page.tsx` before writing. Never delete or replace a non-scaffold route, component tree, research folder, screenshot, or asset namespace unless the user explicitly approves that exact replacement.
- If the planned route already exists, stop and ask whether to update that route, choose another route, or skip it.
- URLs from different origins may require incompatible fonts, global CSS, layouts, and metadata. Before modifying files, ask whether the user wants separate prepared application roots (recommended) or an intentionally combined multi-site app with route-scoped styling. Do not create an unapproved monorepo or silently mix global foundations.

## Pre-Flight

1. **Browser automation is required.** Check for available browser MCP tools (Chrome MCP, Playwright MCP, Browserbase MCP, Puppeteer MCP, etc.). Use whichever is available — if multiple exist, prefer Chrome MCP. If none are detected, ask the user which browser tool they have and how to connect it. This skill cannot work without browser automation.
2. Parse `$ARGUMENTS` as one or more URLs. Normalize and validate each URL; if any are invalid, ask the user to correct them before proceeding. For each valid URL, verify it is accessible via your browser MCP tool.
3. Verify the base project builds: `npm run build`. The Next.js + shadcn/ui + Tailwind v4 scaffold should already be in place. If not, tell the user to set it up first.
4. Inventory existing routes (`src/app/**/page.tsx`), site component namespaces, research artifacts, screenshots, and public assets. Distinguish the untouched template scaffold from existing cloned or user-authored work.
5. Write an output plan listing every target URL, `<app-root>`, `<site-key>`, `<page-key>`, destination route, artifact roots, and whether any shared foundation file must change. Resolve collisions across every planned output, same-path query/fragment behavior, and multi-origin layout decisions with the user before editing.
6. Create only the planned per-page/per-site directories plus `scripts/` if needed. Use unique asset-download script names such as `scripts/download-assets-<site-key>-<page-key>.mjs`; do not overwrite another page's downloader.
7. For multiple pages from one origin, build the shared foundation once, sequentially, before parallel page work. Optionally confirm whether to run page builders in parallel (recommended if resources allow) or sequentially to avoid overload.
8. **Install the local capture and diff harness.** Browser MCP is for interactive probing; it generally cannot save a file to disk or capture below the fold, and it cannot compare two images. Install `playwright` (plus `npx playwright install chromium`) for persisted full-page screenshots, and `pixelmatch` with `pngjs` for the Phase 5 numeric diff. Add all three as devDependencies and tell the user you added them.

## Guiding Principles

These are the truths that separate a successful clone from a "close enough" mess. Internalize them — they should inform every decision you make.

### 1. Completeness Beats Speed

Every builder agent must receive **everything** it needs to do its job perfectly: screenshot, exact CSS values, downloaded assets with local paths, real text content, component structure. If a builder has to guess anything — a color, a font size, a padding value — you have failed at extraction. Take the extra minute to extract one more property rather than shipping an incomplete brief.

### 2. Small Tasks, Perfect Results

When an agent gets "build the entire features section," it glosses over details — it approximates spacing, guesses font sizes, and produces something "close enough" but clearly wrong. When it gets a single focused component with exact CSS values, it nails it every time.

Look at each section and judge its complexity. A simple banner with a heading and a button? One agent. A complex section with 3 different card variants, each with unique hover states and internal layouts? One agent per card variant plus one for the section wrapper. When in doubt, make it smaller.

**Complexity budget rule:** If a builder prompt exceeds ~150 lines of spec content, the section is too complex for one agent. Break it into smaller pieces. This is a mechanical check — don't override it with "but it's all related."

### 3. Real Content, Real Assets

Extract the actual text, images, videos, and SVGs from the live site. This is a clone, not a mockup. Use `element.textContent`, download every `<img>` and `<video>`, extract inline `<svg>` elements as React components. Generate content only when it is clearly server-generated and unique per session, or when the optional Atlas Cloud fallback below is explicitly approved after the original asset proves unrecoverable.

**Layered assets matter.** A section that looks like one image is often multiple layers — a background watercolor/gradient, a foreground UI mockup PNG, an overlay icon. Inspect each container's full DOM tree and enumerate ALL `<img>` elements and background images within it, including absolutely-positioned overlays. Missing an overlay image makes the clone look empty even if the background is correct.

### 4. Foundation First

Nothing can be built until the foundation exists: global CSS with the target site's design tokens (colors, fonts, spacing), TypeScript types for the content structures, and global assets (fonts, favicons). This is sequential and non-negotiable. Everything after this can be parallel.

### 5. Extract How It Looks AND How It Behaves

A website is not a screenshot — it's a living thing. Elements move, change, appear, and disappear in response to scrolling, hovering, clicking, resizing, and time. If you only extract the static CSS of each element, your clone will look right in a screenshot but feel dead when someone actually uses it.

For every element, extract its **appearance** (exact computed CSS via `getComputedStyle()`) AND its **behavior** (what changes, what triggers the change, and how the transition happens). Not "it looks like 16px" — extract the actual computed value. Not "the nav changes on scroll" — document the exact trigger (scroll position, IntersectionObserver threshold, viewport intersection), the before and after states (both sets of CSS values), and the transition (duration, easing, CSS transition vs. JS-driven vs. CSS `animation-timeline`).

Examples of behaviors to watch for — these are illustrative, not exhaustive. The page may do things not on this list, and you must catch those too:
- A navbar that shrinks, changes background, or gains a shadow after scrolling past a threshold
- Elements that animate into view when they enter the viewport (fade-up, slide-in, stagger delays)
- Sections that snap into place on scroll (`scroll-snap-type`)
- Parallax layers that move at different rates than the scroll
- Hover states that animate (not just change — the transition duration and easing matter)
- Dropdowns, modals, accordions with enter/exit animations
- Scroll-driven progress indicators or opacity transitions
- Auto-playing carousels or cycling content
- Dark-to-light (or any theme) transitions between page sections
- **Tabbed/pill content that cycles** — buttons that switch visible card sets with transitions
- **Scroll-driven tab/accordion switching** — sidebars where the active item auto-changes as content scrolls past (IntersectionObserver, NOT click handlers)
- **Smooth scroll libraries** (Lenis, Locomotive Scroll) — check for `.lenis` class or scroll container wrappers

### 6. Identify the Interaction Model Before Building

This is the single most expensive mistake in cloning: building a click-based UI when the original is scroll-driven, or vice versa. Before writing any builder prompt for an interactive section, you must definitively answer: **Is this section driven by clicks, scrolls, hovers, time, or some combination?**

How to determine this:
1. **Don't click first.** Scroll through the section slowly and observe if things change on their own as you scroll.
2. If they do, it's scroll-driven. Extract the mechanism: `IntersectionObserver`, `scroll-snap`, `position: sticky`, `animation-timeline`, or JS scroll listeners.
3. If nothing changes on scroll, THEN click/hover to test for click/hover-driven interactivity.
4. Document the interaction model explicitly in the component spec: "INTERACTION MODEL: scroll-driven with IntersectionObserver" or "INTERACTION MODEL: click-to-switch with opacity transition."

A section with a sticky sidebar and scrolling content panels is fundamentally different from a tabbed interface where clicking switches content. Getting this wrong means a complete rewrite, not a CSS tweak.

### 7. Extract Every State, Not Just the Default

Many components have multiple visual states — a tab bar shows different cards per tab, a header looks different at scroll position 0 vs 100, a card has hover effects. You must extract ALL states, not just whatever is visible on page load.

For tabbed/stateful content:
- Click each tab/button via browser MCP
- Extract the content, images, and card data for EACH state
- Record which content belongs to which state
- Note the transition animation between states (opacity, slide, fade, etc.)

For scroll-dependent elements:
- Capture computed styles at scroll position 0 (initial state)
- Scroll past the trigger threshold and capture computed styles again (scrolled state)
- Diff the two to identify exactly which CSS properties change
- Record the transition CSS (duration, easing, properties)
- Record the exact trigger threshold (scroll position in px, or viewport intersection ratio)

### 8. Spec Files Are the Source of Truth

Every component gets a specification file under that page's artifact root (`docs/research/<site-key>/<page-key>/components/`) BEFORE any builder is dispatched. This file is the contract between your extraction work and the builder agent. The builder receives the spec file contents inline in its prompt — the file also persists as an auditable artifact that the user (or you) can review if something looks wrong.

The spec file is not optional. It is not a nice-to-have. If you dispatch a builder without first writing a spec file, you are shipping incomplete instructions based on whatever you can remember from a browser MCP session, and the builder will guess to fill gaps.

### 9. Build Must Always Compile

Every builder agent must verify `npx tsc --noEmit` passes before finishing. After merging worktrees, you verify `npm run build` passes. A broken build is never acceptable, even temporarily.

## Phase 1: Reconnaissance

Navigate to the target URL with browser MCP.

### Screenshots
- Take **full-page screenshots** at desktop (1440px), tablet (768px), and mobile (390px) viewports
- Save to that page's screenshot root (`docs/design-references/<site-key>/<page-key>/`) with descriptive names
- These are your master reference — builders will receive section-specific crops/screenshots later

Most browser MCP tools return an image into the conversation but cannot write a file to disk, and
many cannot capture beyond the viewport. Use the local Playwright harness from Pre-Flight for the
master references so they persist as artifacts builders and the QA pass can actually open:

```javascript
// scripts/shoot-<site-key>-<page-key>.mjs
import { chromium } from 'playwright';
import { mkdirSync } from 'fs';

const OUT = 'docs/design-references/<site-key>/<page-key>';
const url = process.argv[2];
const tag = process.argv[3];            // 'original' or 'clone'
const viewports = [
  { name: 'desktop-1440', width: 1440, height: 900 },
  { name: 'tablet-768', width: 768, height: 1024 },
  { name: 'mobile-390', width: 390, height: 844 },
];

mkdirSync(OUT, { recursive: true });
const browser = await chromium.launch();
for (const vp of viewports) {
  const ctx = await browser.newContext({ viewport: { width: vp.width, height: vp.height }, deviceScaleFactor: 1 });
  const page = await ctx.newPage();
  await page.goto(url, { waitUntil: 'networkidle', timeout: 90000 });
  // CRITICAL: scroll the whole page first, or every scroll-reveal element
  // screenshots at opacity 0 and the reference is useless
  await page.evaluate(async () => {
    const step = Math.round(window.innerHeight * 0.7);
    for (let y = 0; y < document.body.scrollHeight; y += step) {
      window.scrollTo(0, y);
      await new Promise(r => setTimeout(r, 220));
    }
    window.scrollTo(0, 0);
    await new Promise(r => setTimeout(r, 600));
  });
  await page.waitForTimeout(1200);
  await page.screenshot({ path: `${OUT}/${tag}-${vp.name}-full.png`, fullPage: true });
  await page.screenshot({ path: `${OUT}/${tag}-${vp.name}-hero.png` });
  await ctx.close();
}
await browser.close();
```

Run it once against the live URL with tag `original` now, and again against the local clone with tag
`clone` in Phase 5. Keep browser MCP for interactive probing, where it is the better tool.

### Global Extraction
Extract these from the page before doing anything else:

**Fonts** — Inspect `<link>` tags for Google Fonts or self-hosted fonts. Check computed `font-family` on key elements (headings, body, code, labels). Document every family, weight, and style actually used. For a single-site app, configure shared fonts in `src/app/layout.tsx` using `next/font/google` or `next/font/local`. In an approved combined multi-site app, keep incompatible fonts/layout concerns route-scoped.

**Colors** — Extract the site's color palette from computed styles across the page. For a single-site app, merge the target's colors into `src/app/globals.css` without removing tokens required by existing routes. Map them to shadcn's token names (background, foreground, primary, muted, etc.) where they fit. In an approved combined multi-site app, use a route wrapper or scoped token namespace instead of replacing another site's global palette.

**Favicons & Meta** — Download page/site SEO assets under the planned site asset namespace. Put truly app-global metadata in the root layout only when it applies to every route; otherwise export route-specific metadata from the destination page or a route layout.

**Global UI patterns** — Identify any site-wide CSS or JS: custom scrollbar hiding, scroll-snap on the page container, global keyframe animations, backdrop filters, gradients used as overlays, **smooth scroll libraries** (Lenis, Locomotive Scroll — check for `.lenis`, `.locomotive-scroll`, or custom scroll container classes). Merge truly shared behavior into `globals.css`; keep page-specific behavior scoped to the page so existing routes do not change unexpectedly.

### Read the Shipped Source

Do this BEFORE the interaction sweep. Every site ships its own stylesheets and JavaScript, and they
are readable. Reading them turns guesswork into transcription: instead of probing until a threshold
looks about right, you read the number. Probing tells you a header hides "somewhere around 100px";
the source tells you it hides at `scrollY > 100` and returns after 50px of *accumulated* upward
scroll, which no amount of scrolling would have revealed.

1. **Enumerate the bundles.** Read the asset URLs from the live DOM rather than from a `curl` of the
   HTML, since a server-rendered page often lists different files than the hydrated one:

   ```javascript
   JSON.stringify({
     css: [...document.querySelectorAll('link[rel=stylesheet]')].map(l => new URL(l.href).pathname),
     js: [...document.querySelectorAll('link[rel=modulepreload], script[src]')]
           .map(e => new URL(e.href || e.src).pathname)
   })
   ```

2. **Download them** into your scratchpad, not into the project.

3. **Mine the CSS first.** It is small and dense with exact values:
   - `@theme`, `:root`, or `@layer theme` blocks give the whole design token set (colors, font
     families, type scale, radii, spacing) in one read. Compare each token against your project's
     defaults and override only what actually differs.
   - `@font-face` blocks give real font file URLs, weights, and `font-display`.
   - `@keyframes` blocks give every animation verbatim, including durations and easing.
   - Custom utility classes explain any class name you could not account for.

4. **Mine the JS for behavior.** Grep the downloaded chunks for a distinctive class string you saw
   toggle during the sweep, then read the surrounding code:

   ```bash
   grep -l "translate-y-full" js/*        # find the chunk that owns the header
   ```

   Minified code is still readable when you only need the constants and the control flow. Extract:
   scroll thresholds and their comparison operators, `setInterval` / `setTimeout` durations,
   `IntersectionObserver` thresholds and `rootMargin`, easing formulas, stagger delays, and the exact
   condition behind each state class. Transcribe the logic into your `BEHAVIORS.md` as pseudocode so
   builders implement the rule rather than reverse-engineering it from symptoms.

5. **Check whether the target uses your styling toolkit.** If the DOM is full of Tailwind class
   strings, the markup *is* the style spec. Save each section's comment-stripped `outerHTML` to
   `docs/research/<site-key>/<page-key>/markup/<NN>-<section>.html` and have builders port those
   class strings verbatim instead of re-deriving spacing from `getComputedStyle`. This is faster and
   strictly more accurate. Note that stateful markup needs one capture per state (header at rest and
   scrolled, menu open and closed).

Reading the source never replaces the interaction sweep. The sweep tells you *which* behaviors exist
and which are worth chasing; the source tells you their exact parameters. Do both.

### Mandatory Interaction Sweep

This is a dedicated pass AFTER screenshots and BEFORE anything else. Its purpose is to discover every behavior on the page — many of which are invisible in a static screenshot.

**Scroll sweep:** Scroll the page slowly from top to bottom via browser MCP. At each section, pause and observe:
- Does the header change appearance? Record the scroll position where it triggers.
- Do elements animate into view? Record which ones and the animation type.
- Does a sidebar or tab indicator auto-switch as you scroll? Record the mechanism.
- Are there scroll-snap points? Record which containers.
- Is there a smooth scroll library active? Check for non-native scroll behavior.

**Click sweep:** Click every element that looks interactive:
- Every button, tab, pill, link, card
- Record what happens: does content change? Does a modal open? Does a dropdown appear?
- For tabs/pills: click EACH ONE and record the content that appears for each state

**Hover sweep:** Hover over every element that might have hover states:
- Buttons, cards, links, images, nav items
- Record what changes: color, scale, shadow, underline, opacity

**Responsive sweep:** Test at 3 viewport widths via browser MCP:
- Desktop: 1440px
- Tablet: 768px
- Mobile: 390px
- At each width, note which sections change layout (column → stack, sidebar disappears, etc.) and at approximately which breakpoint the change occurs.

Save all findings to `<artifact-root>/BEHAVIORS.md`. This is your behavior bible — reference it when writing every component spec.

### Page Topology
Map out every distinct section of the page from top to bottom. Give each a working name. Document:
- Their visual order
- Which are fixed/sticky overlays vs. flow content
- The overall page layout (scroll container, column structure, z-index layers)
- Dependencies between sections (e.g., a floating nav that overlays everything)
- **The interaction model** of each section (static, click-driven, scroll-driven, time-driven)

Save this as `<artifact-root>/PAGE_TOPOLOGY.md` — it becomes your assembly blueprint.

## Phase 2: Foundation Build

This is sequential per origin. Do it yourself (not delegated to an agent) since it touches shared files. Re-read the output plan and preserve every existing route before editing:

1. **Merge fonts and shared layout behavior** without deleting requirements of existing routes. Use route layouts when behavior is not truly app-global.
2. **Merge global CSS carefully**; scope page/site-specific tokens, keyframes, scroll behavior, and utilities under a route wrapper when they could conflict.
3. **Create namespaced TypeScript interfaces** for the content structures you've observed; reuse existing same-site types only when their contracts match.
4. **Extract SVG icons** — deduplicate same-site icons under `src/components/sites/<site-key>/shared/icons.tsx`; keep page-only icons in the page component namespace. Name them by visual function (e.g., `SearchIcon`, `ArrowRightIcon`, `LogoIcon`).
5. **Download assets into the planned namespace** — use the page's uniquely named download script and write into `public/sites/<site-key>/<page-key>/` or the approved same-site shared directory. Never write a generic filename over another page's asset.
6. Verify every previously existing route still builds, then run `npm run build`.

### Asset Discovery Script Pattern

Use browser MCP to enumerate all assets on the page:

```javascript
// Run this via browser MCP to discover all assets
JSON.stringify({
  images: [...document.querySelectorAll('img')].map(img => ({
    src: img.src || img.currentSrc,
    alt: img.alt,
    width: img.naturalWidth,
    height: img.naturalHeight,
    // Include parent info to detect layered compositions
    parentClasses: img.parentElement?.className,
    siblings: img.parentElement ? [...img.parentElement.querySelectorAll('img')].length : 0,
    position: getComputedStyle(img).position,
    zIndex: getComputedStyle(img).zIndex
  })),
  videos: [...document.querySelectorAll('video')].map(v => ({
    src: v.src || v.querySelector('source')?.src,
    poster: v.poster,
    autoplay: v.autoplay,
    loop: v.loop,
    muted: v.muted
  })),
  backgroundImages: [...document.querySelectorAll('*')].filter(el => {
    const bg = getComputedStyle(el).backgroundImage;
    return bg && bg !== 'none';
  }).map(el => ({
    url: getComputedStyle(el).backgroundImage,
    element: el.tagName + '.' + el.className?.split(' ')[0]
  })),
  svgCount: document.querySelectorAll('svg').length,
  fonts: [...new Set([...document.querySelectorAll('*')].slice(0, 200).map(el => getComputedStyle(el).fontFamily))],
  favicons: [...document.querySelectorAll('link[rel*="icon"]')].map(l => ({ href: l.href, sizes: l.sizes?.toString() }))
});
```

Then use the uniquely named page download script to fetch everything into its planned asset root. Use batched parallel downloads (4 at a time) with proper error handling.

### Optional Atlas Cloud Fallback for Unrecoverable Visual Assets

This is an exception path, not part of the default clone workflow. Use it only when **all** of the following are true:

- The original asset still cannot be recovered after bounded download attempts and inspection of the rendered page, HTML, CSS, source maps, network responses, and same-site asset paths.
- No lawful local or same-site equivalent is available.
- The asset is not a logo, trademark, product screenshot, legal or certification mark, or other distinctive brand artwork. Those must remain exact originals or be reported as missing.
- The user explicitly approves a generated substitute and understands that it is not pixel-identical source material.
- `ATLASCLOUD_API_KEY` is available from the environment. Never print it, place it in a URL, save it in an artifact, or send it to an output CDN.

When approved, follow this contract:

1. Fetch the live model catalog from `GET https://api.atlascloud.ai/api/v1/models` and choose a currently available `Image` model that supports the required aspect ratio and style. Do not rely on a stale hard-coded model list.
2. Fetch that model's `schema` URL and validate the payload against its current required fields before submitting. `qwen-image-3.0/text-to-image` is an example, not a permanent default.
3. Submit exactly one authenticated `POST https://api.atlascloud.ai/api/v1/model/generateImage` request. Do not automatically retry the generation POST; surface an ambiguous or failed submission to the user.
4. Persist the returned prediction ID in the page's research artifacts, then poll `GET https://api.atlascloud.ai/api/v1/model/prediction/<id>` with bounded backoff (for example, every 3 seconds for at most 40 attempts). Stop immediately on `completed` or `failed`.
5. Accept only HTTPS output URLs from the completed prediction. Download them without the Atlas authorization header, validate the media type and dimensions, and save them under the planned namespaced asset root.
6. Record the model ID, prompt, prediction ID, output path, and the user's approval in `<artifact-root>/ARTIFACT_MANIFEST.md`. Label the file as generated fallback material so builders never treat it as an exact original.

If any condition is not met, keep the missing-asset finding in the artifact manifest and continue without fabricating the source site's identity.

## Phase 3: Component Specification & Dispatch

This is the core loop. For each section in your page topology (top to bottom), you do THREE things: **extract**, **write the spec file**, then **dispatch builders**.

### Step 1: Extract

For each section, use browser MCP to extract everything:

1. **Screenshot** the section in isolation (scroll to it, screenshot the viewport). Save to the page's screenshot root.

2. **Extract CSS** for every element in the section. Use the extraction script below — don't hand-measure individual properties. Run it once per component container and capture the full output:

```javascript
// Per-component extraction — run via browser MCP
// Replace SELECTOR with the actual CSS selector for the component
(function(selector) {
  const el = document.querySelector(selector);
  if (!el) return JSON.stringify({ error: 'Element not found: ' + selector });
  const props = [
    'fontSize','fontWeight','fontFamily','lineHeight','letterSpacing','color',
    'textTransform','textDecoration','backgroundColor','background',
    'padding','paddingTop','paddingRight','paddingBottom','paddingLeft',
    'margin','marginTop','marginRight','marginBottom','marginLeft',
    'width','height','maxWidth','minWidth','maxHeight','minHeight',
    'display','flexDirection','justifyContent','alignItems','gap',
    'gridTemplateColumns','gridTemplateRows',
    'borderRadius','border','borderTop','borderBottom','borderLeft','borderRight',
    'boxShadow','overflow','overflowX','overflowY',
    'position','top','right','bottom','left','zIndex',
    'opacity','transform','transition','cursor',
    'objectFit','objectPosition','mixBlendMode','filter','backdropFilter',
    'whiteSpace','textOverflow','WebkitLineClamp'
  ];
  function extractStyles(element) {
    const cs = getComputedStyle(element);
    const styles = {};
    props.forEach(p => { const v = cs[p]; if (v && v !== 'none' && v !== 'normal' && v !== 'auto' && v !== '0px' && v !== 'rgba(0, 0, 0, 0)') styles[p] = v; });
    return styles;
  }
  function walk(element, depth) {
    if (depth > 4) return null;
    const children = [...element.children];
    return {
      tag: element.tagName.toLowerCase(),
      classes: element.className?.toString().split(' ').slice(0, 5).join(' '),
      text: element.childNodes.length === 1 && element.childNodes[0].nodeType === 3 ? element.textContent.trim().slice(0, 200) : null,
      styles: extractStyles(element),
      images: element.tagName === 'IMG' ? { src: element.src, alt: element.alt, naturalWidth: element.naturalWidth, naturalHeight: element.naturalHeight } : null,
      childCount: children.length,
      children: children.slice(0, 20).map(c => walk(c, depth + 1)).filter(Boolean)
    };
  }
  return JSON.stringify(walk(el, 0), null, 2);
})('SELECTOR');
```

3. **Extract multi-state styles** — for any element with multiple states (scroll-triggered, hover, active tab), capture BOTH states:

```javascript
// State A: capture styles at current state (e.g., scroll position 0)
// Then trigger the state change (scroll, click, hover via browser MCP)
// State B: re-run the extraction script on the same element
// The diff between A and B IS the behavior specification
```

Record the diff explicitly: "Property X changes from VALUE_A to VALUE_B, triggered by TRIGGER, with transition: TRANSITION_CSS."

4. **Extract real content** — all text, alt attributes, aria labels, placeholder text. Use `element.textContent` for each text node. For tabbed/stateful content, **click each tab and extract content per state**.

5. **Identify assets** this section uses — which namespaced downloaded images/videos and which site/page icon components. Check for **layered images** (multiple `<img>` or background-images stacked in the same container).

6. **Assess complexity** — how many distinct sub-components does this section contain? A distinct sub-component is an element with its own unique styling, structure, and behavior (e.g., a card, a nav item, a search panel).

### Step 2: Write the Component Spec File

For each section (or sub-component, if you're breaking it up), create a spec file inside the page's component-spec directory. This is NOT optional — every builder must have a corresponding spec file.

**File path:** `docs/research/<site-key>/<page-key>/components/<component-name>.spec.md`

**Template:**

```markdown
# <ComponentName> Specification

## Overview
- **Target file:** `src/components/sites/<site-key>/<page-key>/<ComponentName>.tsx`
- **Screenshot:** `docs/design-references/<site-key>/<page-key>/<screenshot-name>.png`
- **Interaction model:** <static | click-driven | scroll-driven | time-driven>

## DOM Structure
<Describe the element hierarchy — what contains what>

## Computed Styles (exact values from getComputedStyle)

### Container
- display: ...
- padding: ...
- maxWidth: ...
- (every relevant property with exact values)

### <Child element 1>
- fontSize: ...
- color: ...
- (every relevant property)

### <Child element N>
...

## States & Behaviors

### <Behavior name, e.g., "Scroll-triggered floating mode">
- **Trigger:** <exact mechanism — scroll position 50px, IntersectionObserver rootMargin "-30% 0px", click on .tab-button, hover>
- **State A (before):** maxWidth: 100vw, boxShadow: none, borderRadius: 0
- **State B (after):** maxWidth: 1200px, boxShadow: 0 4px 20px rgba(0,0,0,0.1), borderRadius: 16px
- **Transition:** transition: all 0.3s ease
- **Implementation approach:** <CSS transition + scroll listener | IntersectionObserver | CSS animation-timeline | etc.>

### Hover states
- **<Element>:** <property>: <before> → <after>, transition: <value>

## Per-State Content (if applicable)

### State: "Featured"
- Title: "..."
- Subtitle: "..."
- Cards: [{ title, description, image, link }, ...]

### State: "Productivity"
- Title: "..."
- Cards: [...]

## Assets
- Background image: `public/sites/<site-key>/<page-key>/images/<file>.webp`
- Overlay image: `public/sites/<site-key>/<page-key>/images/<file>.png`
- Icons used: <ArrowIcon>, <SearchIcon> from the planned page or same-site shared icon module

## Text Content (verbatim)
<All text content, copy-pasted from the live site>

## Responsive Behavior
- **Desktop (1440px):** <layout description>
- **Tablet (768px):** <what changes — e.g., "maintains 2-column, gap reduces to 16px">
- **Mobile (390px):** <what changes — e.g., "stacks to single column, images full-width">
- **Breakpoint:** layout switches at ~<N>px
```

Fill every section. If a section doesn't apply (e.g., no states for a static footer), write "N/A" — but think twice before marking States & Behaviors as N/A. Even a footer might have hover states on links.

### Step 3: Dispatch Builders

Based on complexity, dispatch builder agent(s) in worktree(s):

**Simple section** (1-2 sub-components): One builder agent gets the entire section.

**Complex section** (3+ distinct sub-components): Break it up. One agent per sub-component, plus one agent for the section wrapper that imports them. Sub-component builders go first since the wrapper depends on them.

**What every builder agent receives:**
- The full contents of its component spec file (inline in the prompt — don't say "go read the spec file")
- Path to the section screenshot in the page's namespaced screenshot root
- Which shared components to import (the planned site-scoped icon module, `cn()`, shadcn primitives)
- The namespaced target file path (e.g., `src/components/sites/<site-key>/<page-key>/HeroSection.tsx`)
- Instruction to verify with `npx tsc --noEmit` before finishing
- For responsive behavior: the specific breakpoint values and what changes

**Don't wait.** As soon as you've dispatched the builder(s) for one section, move to extracting the next section. Builders work in parallel in their worktrees while you continue extraction.

### Step 4: Merge

As builder agents complete their work:
- Merge their worktree branches into main
- You have full context on what each agent built, so resolve any conflicts intelligently
- Reject or repair any merge that deletes or rewrites an unrelated existing route or another page's namespace
- After each merge, verify the build still passes: `npm run build`
- If a merge introduces type errors, fix them immediately

The extract → spec → dispatch → merge cycle continues until all sections are built.

## Phase 4: Page Assembly

After all sections are built and merged, wire the page into the exact destination route from the approved output plan. Use `src/app/page.tsx` only for the first fresh-template root clone; otherwise use the planned path such as `src/app/docs/intro/page.tsx`:

- Import all section components
- Implement the page-level layout from your topology doc (scroll containers, column structures, sticky positioning, z-index layering)
- Connect real content to component props
- Implement page-level behaviors: scroll snap, scroll-driven animations, dark-to-light transitions, intersection observers, smooth scroll (Lenis etc.)
- Confirm all routes that existed before this run are still present and were not unintentionally changed
- Verify: `npm run build` passes clean

## Phase 5: Visual QA Diff

After assembly, do NOT declare the clone complete. Measure the difference, do not eyeball it. Eyes
miss a 4px padding error and waste attention on photo compression noise; the two checks below invert
that. Run them at 1440px, 768px, and 390px.

### 5a. Layout alignment (run this first)

The cheapest and highest-signal check. Measure the vertical position of a marker in every section on
both sites and diff the numbers. A single mismatch localizes the fault to one section instantly,
and identical totals prove the whole page is aligned.

```javascript
// run in each page via Playwright, after scrolling the page to fire scroll reveals
const MARKERS = ['About us', 'Our services', /* one distinctive string per section */];
await page.evaluate((markers) => {
  const res = {};
  for (const m of markers) {
    const el = [...document.querySelectorAll('h1,h2,h3,p,span,a')]
      .find(e => (e.textContent || '').trim().startsWith(m));
    res[m] = el ? Math.round(el.getBoundingClientRect().top + window.scrollY) : null;
  }
  res.__docHeight = document.documentElement.scrollHeight;
  return res;
}, MARKERS);
```

Print `original`, `clone`, and `delta` per marker. Every delta should be `0`; treat 1px as
sub-pixel rounding. **Read the deltas top to bottom and fix the first non-zero one before looking at
the rest**, since one wrong margin shifts everything below it and inflates the whole list.

A delta that appears only at the first section carrying a `margin-top` usually means a margin
collapsed in one version and not the other. Compare how the source roots its sections: a wrapper
that is `display: contents` or a flex container makes every section a flex item, and flex items
never collapse margins.

### 5b. Automated pixel diff

Screenshot both pages into aligned bands, then diff them numerically.

```bash
npm i -D pixelmatch pngjs
```

```javascript
// scripts/pixeldiff-<site-key>-<page-key>.mjs
import pixelmatch from 'pixelmatch';
import { PNG } from 'pngjs';
import { readFileSync, writeFileSync } from 'fs';

const a = PNG.sync.read(readFileSync(`${DIR}/${band}-original.png`));
const b = PNG.sync.read(readFileSync(`${DIR}/${band}-clone.png`));
const diff = new PNG({ width: a.width, height: a.height });
const n = pixelmatch(a.data, b.data, diff.data, a.width, a.height, { threshold: 0.12 });
const pct = (n / (a.width * a.height)) * 100;
if (pct > 0.5) writeFileSync(`${DIR}/${band}-DIFF.png`, PNG.sync.write(diff));
```

Capture the bands with Playwright using `fullPage: true` plus a `clip` of
`{ x: 0, y: i * 900, width, height: 900 }`, one file per band per target.

Reading the numbers:
- **Under ~0.5%**: anti-aliasing and image encoding noise. Ignore.
- **0.5% to a few percent**: open the written `-DIFF.png`. Red confined to the inside of a photo is
  encoder noise or an animation caught mid-cycle, both acceptable. Red on text, edges, borders, or
  buttons is a real defect.
- **Uniform red over a whole text block** that measures correct in 5a is a 1px shift, not a
  restyle. Confirm with 5a before touching any CSS.
- **A band that is red everywhere**: usually a slideshow or video on a different frame. Confirm by
  checking which asset each side is displaying before assuming a bug.

Only open a diff image when the number says something is wrong. That is the point of measuring.

### 5c. Fix and re-verify

1. For each real discrepancy, check the component spec file: was the value extracted correctly?
   - If the spec was wrong: re-extract, update the spec, then fix the component
   - If the spec was right but the builder got it wrong: fix the component to match the spec
2. Re-run 5a and 5b after every fix.
3. Test all interactive behaviors: scroll through the page, click every button/tab, hover over
   interactive elements. Script this too, asserting the exact values from `BEHAVIORS.md`, for
   example that the header carries `-translate-y-full` past its threshold and that a counter lands
   on its real final value.
4. Verify smooth scroll feels right, header transitions work, tab switching works, animations play.

Only after this visual QA pass is the clone complete.

## Pre-Dispatch Checklist

Before dispatching ANY builder agent, verify you can check every box. If you can't, go back and extract more.

- [ ] Spec file written to `docs/research/<site-key>/<page-key>/components/<name>.spec.md` with ALL sections filled
- [ ] Every CSS value in the spec is from `getComputedStyle()`, not estimated
- [ ] Interaction model is identified and documented (static / click / scroll / time)
- [ ] For stateful components: every state's content and styles are captured
- [ ] For scroll-driven components: trigger threshold, before/after styles, and transition are recorded
- [ ] For hover states: before/after values and transition timing are recorded
- [ ] All images in the section are identified (including overlays and layered compositions)
- [ ] Responsive behavior is documented for at least desktop and mobile
- [ ] Text content is verbatim from the site, not paraphrased
- [ ] The builder prompt is under ~150 lines of spec; if over, the section needs to be split

## What NOT to Do

These are lessons from previous failed clones — each one cost hours of rework:

- **Don't build click-based tabs when the original is scroll-driven (or vice versa).** Determine the interaction model FIRST by scrolling before clicking. This is the #1 most expensive mistake — it requires a complete rewrite, not a CSS fix.
- **Don't extract only the default state.** If there are tabs showing "Featured" on load, click Productivity, Creative, Lifestyle and extract each one's cards/content. If the header changes on scroll, capture styles at position 0 AND position 100+.
- **Don't miss overlay/layered images.** A background watercolor + foreground UI mockup = 2 images. Check every container's DOM tree for multiple `<img>` elements and positioned overlays.
- **Don't build mockup components for content that's actually videos/animations.** Check if a section uses `<video>`, Lottie, or canvas before building elaborate HTML mockups of what the video shows.
- **Don't approximate CSS classes.** "It looks like `text-lg`" is wrong if the computed value is `18px` and `text-lg` is `18px/28px` but the actual line-height is `24px`. Extract exact values.
- **Don't build everything in one monolithic commit.** The whole point of this pipeline is incremental progress with verified builds at each step.
- **Don't treat a new target as permission to replace the current app.** Preserve existing routes and namespaced artifacts; ask before updating a route that already exists.
- **Don't reference docs from builder prompts.** Each builder gets the CSS spec inline in its prompt — never "see DESIGN_TOKENS.md for colors." The builder should have zero need to read external docs.
- **Don't skip asset extraction.** Without real images, videos, and fonts, the clone will always look fake regardless of how perfect the CSS is.
- **Don't give a builder agent too much scope.** If you're writing a builder prompt and it's getting long because the section is complex, that's a signal to break it into smaller tasks.
- **Don't bundle unrelated sections into one agent.** A CTA section and a footer are different components with different designs — don't hand them both to one agent and hope for the best.
- **Don't skip responsive extraction.** If you only inspect at desktop width, the clone will break at tablet and mobile. Test at 1440, 768, and 390 during extraction.
- **Don't forget smooth scroll libraries.** Check for Lenis (`.lenis` class), Locomotive Scroll, or similar. Default browser scrolling feels noticeably different and the user will spot it immediately.
- **Don't dispatch builders without a spec file.** The spec file forces exhaustive extraction and creates an auditable artifact. Skipping it means the builder gets whatever you can fit in a prompt from memory.

## Completion

When done, report:
- Source URL to destination-route mapping for every page built
- Existing routes preserved and any explicitly approved replacements
- Total sections built
- Total components created
- Total spec files written (should match components)
- Total assets downloaded (images, videos, SVGs, fonts)
- Build status (`npm run build` result)
- Visual QA results (any remaining discrepancies)
- Any known gaps or limitations

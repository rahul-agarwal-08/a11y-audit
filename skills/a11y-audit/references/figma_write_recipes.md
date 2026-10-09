# Figma write recipes for `a11y-audit`

Patterns to adapt against the live file, not copy-paste-verbatim scripts. Node structure varies between files, so check that each step landed where expected before building on top of it.

**Output shape:** one review page → a "Start here" overview at the top → one Figma **Section** per check group, stacked in group order. Each section is self-contained: its own copy of the reference screenshot, its own numbered callouts, its own column of suggestion cards.

## Platform gotchas

- Always `await figma.setCurrentPageAsync(page)` to change pages — setting `figma.currentPage` directly is **not supported**.
- Load every font/style a call uses (Inter **Regular**, **Semi Bold**, **Bold**) *in that same call*, before setting `.characters`. Use `loadFonts()` from the shared block.
- Never call `loadAllPagesAsync`, `setPluginData`, or `createImageAsync` — not supported here. The synchronous `figma.createImage(bytes)` **does** work, and pairs with `node.exportAsync()` (see Step B).
- `figma.createAutoLayout(direction)` takes **only** a direction. A second options argument (e.g. `{ name }`) is silently ignored — set `.name` on the next line.
- Nodes on a page that isn't loaded return empty from `findAll`. Before reading inside the original frame, walk up to its page and `await page.loadAsync()`. That's why section counts are stored in section **names** (see Step H).
- For Inter, the style strings are `"Semi Bold"` and `"Extra Bold"`, not `"SemiBold"`/`"ExtraBold"`.
- Each `use_figma` call is a separate script context. Re-declare the shared block and re-locate nodes **by name** at the start of every call.
- **Children of a Section are positioned relative to the Section**, like children of a frame. Append a node to the section first, then set its `x`/`y` in section-local coordinates. All anchor and callout math below is section-local.
- Sections can't live inside auto-layout frames; they sit directly on the page. Resize them with `section.resizeWithoutConstraints(w, h)`.
- `figma.createVector()` + `vectorPaths` auto-fits the node's `x`/`y`/`width`/`height` to the path's bounding box. Append the vector to the section **before** setting `vectorPaths`, and author the path in section-local coordinates. Screenshot after the first leader line; if it lands offset, read back the node's `x`/`y` and correct the path data rather than guessing twice.
- `layoutWrap: 'WRAP'` needs a newer API version. Guard it with `if ('layoutWrap' in node) node.layoutWrap = 'WRAP';` — older versions just keep one row, which is harmless.
- `node.clone()` of a hidden node returns a hidden clone; set `visible = true` on the copy.
- No text on the review page below 12px, and every text/fill pair must clear 4.5:1. The palette below is pre-checked.

## Shared block — paste at the top of every `use_figma` call

Keep it **byte-identical** across calls, so a section's badges, card pills and overview row are always the same color.

```js
const MIN_TEXT = 12;
const WHITE = { r: 1, g: 1, b: 1 };
const INK   = { r: 0.15, g: 0.15, b: 0.15 };
const MUTED = { r: 0.40, g: 0.40, b: 0.40 };   // 5.38:1 on the card/section tints used below

// One color per check group. White text on every one of these clears 4.5:1 (ratios in comments).
const GROUPS = [
  { n: 1, key: 'names',     title: 'Accessible names', kind: 'Blueline', blueline: 'Accessible name',
    blurb: 'Names for icons, images and controls that have no visible text label.',
    color: { r: 0.08, g: 0.38, b: 0.80 } },   // 5.82:1
  { n: 2, key: 'headings',  title: 'Headings',         kind: 'Blueline', blueline: 'Heading',
    blurb: 'Heading levels that give the screen a clear outline for screen reader users.',
    color: { r: 0.45, g: 0.25, b: 0.75 } },   // 6.55:1
  { n: 3, key: 'landmarks', title: 'Landmarks',        kind: 'Blueline', blueline: 'Landmark',
    blurb: 'Page regions that screen reader users can jump between.',
    color: { r: 0.00, g: 0.45, b: 0.45 } },   // 5.70:1
  { n: 4, key: 'focus',     title: 'Focus order',      kind: 'Blueline', blueline: 'Focus order',
    blurb: 'The order keyboard users move through interactive elements with Tab.',
    color: { r: 0.70, g: 0.15, b: 0.45 } },   // 6.12:1
  { n: 5, key: 'contrast',  title: 'Color & contrast', kind: 'Design',
    blurb: 'Text and UI colors that would read more clearly with a small adjustment.',
    color: { r: 0.72, g: 0.33, b: 0.05 } },   // 4.88:1
  { n: 6, key: 'targets',   title: 'Touch targets',    kind: 'Design',
    blurb: 'Controls whose hit area could grow to at least 24 × 24px.',
    color: { r: 0.10, g: 0.45, b: 0.20 } },   // 5.95:1
  { n: 7, key: 'text',      title: 'Text & links',     kind: 'Design',
    blurb: 'Link wording, text size and text alignment.',
    color: { r: 0.35, g: 0.35, b: 0.45 } },   // 6.75:1
  { n: 8, key: 'forms',     title: 'Forms & errors',   kind: 'Design',
    blurb: 'Labels, instructions and error messages for inputs.',
    color: { r: 0.55, g: 0.30, b: 0.10 } },   // 6.60:1
  { n: 9, key: 'devqa',     title: 'For dev & QA',     kind: 'QA',
    blurb: 'Things a static design can’t show — worth confirming in the build.',
    color: { r: 0.30, g: 0.30, b: 0.30 } },   // 8.52:1
];
const groupBy = key => GROUPS.find(g => g.key === key);

// Chip palettes [background, text] — all >= 4.5:1
const NEUTRAL_CHIP = [{ r: 0.93, g: 0.93, b: 0.93 }, { r: 0.35, g: 0.35, b: 0.35 }];  // 5.97:1
const STATUS_CHIP = {
  'Looks good':         [{ r: 0.85, g: 0.95, b: 0.87 }, { r: 0.10, g: 0.45, b: 0.20 }],  // 5.02:1
  'Not on this screen': [{ r: 0.92, g: 0.92, b: 0.92 }, { r: 0.40, g: 0.40, b: 0.40 }],  // 4.80:1
  'Confirm in build':   [{ r: 0.88, g: 0.92, b: 1.00 }, { r: 0.15, g: 0.30, b: 0.60 }],  // 6.74:1
};

async function loadFonts() {
  for (const style of ["Regular", "Semi Bold", "Bold"]) {
    await figma.loadFontAsync({ family: "Inter", style });
  }
}

// Find-or-create the review page and switch to it. First thing in every call.
async function ensureReviewPage(frameName) {
  const name = `Accessibility head start — ${frameName}`;
  let page = figma.root.children.find(p => p.name === name);
  if (!page) { page = figma.createPage(); page.name = name; }
  await figma.setCurrentPageAsync(page);
  return page;
}

const isAutoLayout = n => 'layoutMode' in n && n.layoutMode !== 'NONE';

function addText(parent, characters, size, weight = "Regular", color = INK) {
  const t = figma.createText();
  t.fontName = { family: "Inter", style: weight };
  t.fontSize = Math.max(size, MIN_TEXT);
  t.characters = characters;
  t.fills = [{ type: "SOLID", color }];
  parent.appendChild(t);
  if (isAutoLayout(parent)) { t.layoutSizingHorizontal = 'FILL'; t.textAutoResize = 'HEIGHT'; }
  return t;
}

function chip(parent, label, bg, fg) {
  const c = figma.createAutoLayout('HORIZONTAL'); c.name = `Chip: ${label}`;
  c.paddingLeft = c.paddingRight = 8;
  c.paddingTop = c.paddingBottom = 3;
  c.cornerRadius = 10;
  c.counterAxisAlignItems = 'CENTER';
  c.fills = [{ type: "SOLID", color: bg }];
  parent.appendChild(c);
  const t = figma.createText();
  t.fontName = { family: "Inter", style: "Semi Bold" };
  t.fontSize = MIN_TEXT;
  t.characters = label;
  t.fills = [{ type: "SOLID", color: fg }];
  c.appendChild(t);
  return c;
}

// Numbered pill in a group's color. Used for canvas badges and for card headers, so they match.
function numberPill(parent, label, color) {
  const p = figma.createAutoLayout('HORIZONTAL'); p.name = `Badge ${label}`;
  p.paddingLeft = p.paddingRight = 8;
  p.paddingTop = p.paddingBottom = 4;
  p.cornerRadius = 12;
  p.primaryAxisAlignItems = 'CENTER';
  p.counterAxisAlignItems = 'CENTER';
  p.fills = [{ type: "SOLID", color }];
  parent.appendChild(p);
  p.minWidth = 24;   // equal-width pills for single digits keep overview rows aligned
  const t = figma.createText();
  t.fontName = { family: "Inter", style: "Bold" };
  t.fontSize = MIN_TEXT;
  t.characters = String(label);
  t.fills = [{ type: "SOLID", color: WHITE }];
  p.appendChild(t);
  return p;
}
```

## Step A — Review page

Call `await ensureReviewPage(frameName)` at the top of every `use_figma` call. It's idempotent, so later one-at-a-time runs in the same conversation land on the same page. For image input, the new file's page is the review page; rename it with the same convention.

## Step B — The master reference screenshot (once per conversation)

**Primary method: export inside Figma (no network needed).** In one `use_figma` call, load the original frame's page, `exportAsync` the frame (read-only), turn the bytes into an image with `figma.createImage`, and fill a rectangle on the review page with it. This avoids the `upload_assets` round trip, which needs network access to Figma's upload service and has a 400×300 distortion bug.

```js
const page = await ensureReviewPage(frameName);
let master = page.findOne(n => n.name === 'Reference screenshot — master');
if (!master) {
  const src = await figma.getNodeByIdAsync(ORIGINAL_FRAME_ID);          // read-only
  let sp = src; while (sp && sp.type !== 'PAGE') sp = sp.parent;
  if (sp && sp.loadAsync) await sp.loadAsync();
  const bytes = await src.exportAsync({ format: 'PNG', constraint: { type: 'SCALE', value: 2 } });
  const image = figma.createImage(bytes);
  master = figma.createRectangle();
  page.appendChild(master);
  master.resize(src.width, src.height);                                  // 1 frame unit = 1px keeps anchor math simple
  master.fills = [{ type: 'IMAGE', imageHash: image.hash, scaleMode: 'FILL' }];
  master.name = 'Reference screenshot — master';
  master.x = -4000; master.y = 0; master.visible = false;              // hidden template; every section clones it
}
```

**Fallback: `get_screenshot` + `upload_assets`**, only if `exportAsync` isn't available:
1. `get_screenshot` on the original frame with a `maxDimension` high enough to keep small elements legible (1600–2000).
2. A `use_figma` call that does only `ensureReviewPage(...)`, so the upload lands on the review page.
3. `upload_assets` with the PNG and no `nodeId`. The response includes `placedOnNodeId`.
4. **Known gotcha:** `upload_assets` places the image in a fixed 400×300 frame regardless of aspect ratio, so resize the placed node to the frame's aspect ratio in a follow-up call.
5. Rename it `Reference screenshot — master`, move it to `x = -4000` and set `visible = false`.

```js
const master = page.findOne(n => n.name === 'Reference screenshot — master');
```

## Read-only measurement (inside component instances)

`get_metadata` doesn't expand instances, so nested icon buttons have no coordinates there. Measure them with one read-only script that writes no properties, removes nothing and creates nothing:

```js
const frame = await figma.getNodeByIdAsync(ORIGINAL_FRAME_ID);
let sp = frame; while (sp && sp.type !== 'PAGE') sp = sp.parent; if (sp && sp.loadAsync) await sp.loadAsync();
const fb = frame.absoluteBoundingBox;
const want = /Action button|Button|Icon|Radio|Text field|Link/i;              // adjust to the file's layer naming
return frame.findAll(n => n.visible && want.test(n.name) && n.absoluteBoundingBox).map(n => {
  const b = n.type === 'TEXT' && n.absoluteRenderBounds ? n.absoluteRenderBounds : n.absoluteBoundingBox;
  return { id: n.id, name: n.name, type: n.type, x: Math.round(b.x - fb.x), y: Math.round(b.y - fb.y), w: Math.round(b.width), h: Math.round(b.height) };
});
```

For **text**, always use `absoluteRenderBounds` (the glyphs) rather than the container's box. In testing, an anchor placed from the container landed in the middle of the word "at" and read like a strikethrough. Find text nodes through their parent (`header.findOne(n => n.type === 'TEXT')`) rather than matching `characters` exactly, which can fail on invisible characters.

## Step C — Create one check section

Re-running a check replaces that check's own section (a node this skill created), so the page never accumulates duplicates.

```js
const LANE = 220;        // clear margin around the screenshot copy, where callout badges live
const HEADER_H = 120;    // room for the section's own title block
const STACK_GAP = 160;   // vertical gap between sections

const checkSections = page =>
  page.children.filter(n => n.type === 'SECTION' && /^\d+ · /.test(n.name));

function createCheckSection(page, g, master) {
  const old = checkSections(page).find(s => s.name.startsWith(`${g.n} · `));
  if (old) old.remove();

  const section = figma.createSection();
  section.name = `${g.n} · ${g.title}`;               // count is appended in Step H
  section.fills = [{ type: "SOLID", color: { r: 0.98, g: 0.98, b: 0.98 } }];
  section.x = 0;
  section.y = 0;                                       // final position set by restack() in Step H

  // Title block: color bar + title + one-line description
  const bar = figma.createRectangle();
  section.appendChild(bar);
  bar.resize(6, 64); bar.x = 48; bar.y = 40;
  bar.fills = [{ type: "SOLID", color: g.color }];
  const title = addText(section, `${g.n} · ${g.title}`, 28, "Bold");
  title.x = 68; title.y = 36;
  const blurb = addText(section, g.blurb, 14, "Regular", MUTED);
  blurb.x = 68; blurb.y = 78;

  // Screenshot copy — a clone of a node this skill created, never the original frame
  const img = master.clone();
  section.appendChild(img);
  img.visible = true;
  img.name = 'Reference screenshot (static copy of the audited frame)';
  img.x = LANE; img.y = HEADER_H + LANE;

  return { section, img };
}
```

Record `img.x`, `img.y`, `img.width` and `img.height` (section-local) for Steps D and E.

## Step D — Compute anchor positions (section-local)

The **anchor** is the exact spot on the screenshot copy where a leader line touches down. Use the original frame's absolute bounding box (read-only, from `get_metadata`) and the screenshot copy's section-local box:

```
scale   = img.width / originalFrame.width
anchorX = img.x + (elementX - originalFrame.x) * scale
anchorY = img.y + (elementY - originalFrame.y) * scale
```

**Anchor on the side of the element that faces its badge**, not its center: a line from the right-hand margin touches the element's right edge, a line from the top margin its top edge. Center anchors make lines cross neighbouring controls; in testing, a line to the Discard icon ran through the Confirm button first. Choose the lane deliberately and set `f.edge` yourself when the nearest edge would route a line across other UI, because `layOutCallouts` keeps a preset `edge`. For text, anchor just past the glyphs (`absoluteRenderBounds` + 6px).

**For elements missing from the metadata** (large files often return unexpanded `<instance>` nodes), estimate from the screenshot: read the element's pixel position, convert with `frameCoord = screenshotPixel / (screenshotWidth / originalFrameWidth)`, then apply the formula above. Don't substitute a pattern-based guess ("nth item × row height"); only the visual read is grounded in what's on screen. Expect roughly 1 in 15–20 visual estimates to need a nudge after the first render — that's normal.

**Focus order:** one anchor per tab stop. A composite widget (toolbar, tab strip, tree) gets **one** anchor and one badge, with the arrow-key note on its card.

## Step E — Callouts: leader line to a numbered badge in the margin

**The badge never sits on top of the UI.** A thin line runs from the anchor out to the margin, and the numbered badge sits at the far end. Badges are numbered 1, 2, 3… within each section, in the section's color, matching the card numbers. In Focus order, the number is the tab stop.

Group anchors by nearest image edge, sort along that edge, and lay them out with three rules:

1. **Lanes stay within the image's own extent** (top/bottom lanes in `[img.x, img.x + img.width]`, left/right in `[img.y, img.y + img.height]`), so badges never drift into corner zones. Overflow spills into a **second ring** further out.
2. **Pitch accounts for pill shape** — wider than tall, so horizontal lanes need more spacing.
3. **A final overlap pass** pushes any badges that still intersect out by one ring.

```js
const CALLOUT_GAP = 56;   // image edge → first ring
const RING_STEP   = 40;   // each extra ring
const PITCH_H     = 46;   // top/bottom lanes
const PITCH_V     = 34;   // left/right lanes

function assignEdge(f, img) {
  const d = {
    top: f.ay - img.y,
    bottom: (img.y + img.height) - f.ay,
    left: f.ax - img.x,
    right: (img.x + img.width) - f.ax,
  };
  return Object.keys(d).reduce((a, b) => (d[a] < d[b] ? a : b));
}

function layOutLane(list, axisKey, pitch, min, max) {
  const capacity = Math.max(1, Math.floor((max - min) / pitch) + 1);
  for (let start = 0; start < list.length; start += capacity) {
    const ring = list.slice(start, start + capacity);
    ring.forEach(f => { f.ring = start / capacity; });
    ring.forEach((f, i) => { f.pos = i === 0 ? f[axisKey] : Math.max(f[axisKey], ring[i - 1].pos + pitch); });
    for (let i = ring.length - 1; i >= 0; i--) {
      const limit = i === ring.length - 1 ? max : ring[i + 1].pos - pitch;
      if (ring[i].pos > limit) ring[i].pos = limit;
    }
  }
}

function placeBadge(f, img) {
  const gap = CALLOUT_GAP + (f.ring || 0) * RING_STEP;
  if (f.edge === 'top')    { f.bx = f.pos;                   f.by = img.y - gap; }
  if (f.edge === 'bottom') { f.bx = f.pos;                   f.by = img.y + img.height + gap; }
  if (f.edge === 'left')   { f.bx = img.x - gap;             f.by = f.pos; }
  if (f.edge === 'right')  { f.bx = img.x + img.width + gap; f.by = f.pos; }
}

function resolveOverlaps(items, img) {
  for (let pass = 0; pass < 4; pass++) {
    let moved = false;
    for (let i = 0; i < items.length; i++) {
      for (let j = i + 1; j < items.length; j++) {
        const a = items[i], b = items[j];
        if (Math.abs(a.bx - b.bx) < PITCH_H && Math.abs(a.by - b.by) < PITCH_V) {
          b.ring = (b.ring || 0) + 1;
          placeBadge(b, img);
          moved = true;
        }
      }
    }
    if (!moved) break;
  }
}

function layOutCallouts(items, img) {   // items = [{ n, ax, ay, edge? }, ...] — section-local anchors
  const lanes = { top: [], bottom: [], left: [], right: [] };
  for (const f of items) { f.edge = f.edge || assignEdge(f, img); lanes[f.edge].push(f); }   // a preset edge wins
  lanes.top.sort((a, b) => a.ax - b.ax);
  lanes.bottom.sort((a, b) => a.ax - b.ax);
  lanes.left.sort((a, b) => a.ay - b.ay);
  lanes.right.sort((a, b) => a.ay - b.ay);
  layOutLane(lanes.top,    'ax', PITCH_H, img.x, img.x + img.width);
  layOutLane(lanes.bottom, 'ax', PITCH_H, img.x, img.x + img.width);
  layOutLane(lanes.left,   'ay', PITCH_V, img.y, img.y + img.height);
  layOutLane(lanes.right,  'ay', PITCH_V, img.y, img.y + img.height);
  for (const f of items) placeBadge(f, img);
  resolveOverlaps(items, img);
  return items;
}

function createCallout(section, f, color) {
  const line = figma.createVector();
  section.appendChild(line);                    // append first, then set geometry (section-local)
  line.vectorPaths = [{ windingRule: "NONE", data: `M ${f.ax} ${f.ay} L ${f.bx} ${f.by}` }];
  line.strokes = [{ type: "SOLID", color }];
  line.strokeWeight = 1.5;
  line.name = "Leader line";

  const badge = numberPill(section, f.n, color);
  badge.x = f.bx - badge.width / 2;
  badge.y = f.by - badge.height / 2;

  const grp = figma.group([line, badge], section);
  grp.name = `Callout ${f.n}`;
  return grp;
}
```

`(f.bx, f.by)` is always outside the screenshot copy. If a lane still produces a crossed or awkward line near a dense corner, override that one item's `bx`/`by` to a cleaner spot on the same edge and redraw — verify and nudge, same as anchor corrections.

## Step F — The suggestion column

Placed to the right of the outermost callout ring. Every card reads in the same order: **the suggestion → why it helps → details**. The title is the suggested value itself, so a designer can scan a column of titles and already have most of their annotations.

```js
function suggestionColumn(section, x, y) {
  const col = figma.createAutoLayout('VERTICAL'); col.name = 'Suggestions';
  section.appendChild(col);
  col.x = x; col.y = y;
  col.itemSpacing = 14;
  col.fills = [];
  col.resize(560, col.height);
  col.layoutSizingHorizontal = 'FIXED';
  return col;
}

function subhead(col, text) {
  const t = addText(col, text, 13, "Bold", MUTED);
  return t;
}

function suggestionCard(col, s, g) {
  // s = { n, suggestion, element, why, sc, jira, confidence, priority?, note? }
  const card = figma.createAutoLayout('VERTICAL'); card.name = `Suggestion ${s.n}`;
  card.itemSpacing = 8;
  card.paddingLeft = card.paddingRight = card.paddingTop = card.paddingBottom = 16;
  card.cornerRadius = 8;
  card.fills = [{ type: "SOLID", color: WHITE }];
  card.strokes = [{ type: "SOLID", color: { r: 0.86, g: 0.86, b: 0.86 } }];
  card.strokeWeight = 1;
  card.strokeAlign = 'INSIDE';
  col.appendChild(card);
  card.layoutSizingHorizontal = 'FILL';

  // 1. The suggestion
  const head = figma.createAutoLayout('HORIZONTAL'); head.name = 'Suggestion';
  head.itemSpacing = 10;
  head.counterAxisAlignItems = 'CENTER';
  head.fills = [];
  card.appendChild(head);
  head.layoutSizingHorizontal = 'FILL';
  numberPill(head, s.n, g.color);
  addText(head, s.suggestion, 15, "Bold");
  addText(card, s.element, MIN_TEXT, "Regular", MUTED);

  // 2. Why it helps
  addText(card, s.why, 13, "Regular");
  if (s.note) addText(card, s.note, MIN_TEXT, "Regular", MUTED);   // e.g. arrow-key note, "if X, use Y"

  // 3. Details
  const chips = figma.createAutoLayout('HORIZONTAL'); chips.name = 'Details';
  chips.itemSpacing = 6;
  chips.fills = [];
  card.appendChild(chips);
  chips.layoutSizingHorizontal = 'FILL';
  if ('layoutWrap' in chips) { chips.layoutWrap = 'WRAP'; chips.counterAxisSpacing = 6; }
  chip(chips, g.kind === 'Blueline' ? `Blueline: ${g.blueline}` : 'Design suggestion', g.color, WHITE);
  if (s.priority) chip(chips, `${s.priority} priority`, NEUTRAL_CHIP[0], NEUTRAL_CHIP[1]);
  chip(chips, s.sc, NEUTRAL_CHIP[0], NEUTRAL_CHIP[1]);
  chip(chips, s.jira, NEUTRAL_CHIP[0], NEUTRAL_CHIP[1]);
  chip(chips, `${s.confidence} confidence`, NEUTRAL_CHIP[0], NEUTRAL_CHIP[1]);
  return card;
}

// "Looks good", "Not on this screen", "Confirm in build" — one compact row each
function statusRow(col, status, text) {
  const row = figma.createAutoLayout('HORIZONTAL'); row.name = `Status: ${status}`;
  row.itemSpacing = 8;
  row.counterAxisAlignItems = 'CENTER';
  row.fills = [];
  col.appendChild(row);
  row.layoutSizingHorizontal = 'FILL';
  const [bg, fg] = STATUS_CHIP[status];
  chip(row, status, bg, fg);
  const t = addText(row, text, MIN_TEXT, "Regular");
  return row;
}
```

Column x = right edge of the outermost badge on the right side (or `img.x + img.width + CALLOUT_GAP` if none) + 80. Column y = `img.y`.

Build the column in this order:
1. One `suggestionCard()` per suggestion, in badge-number order.
2. `subhead(col, "Checked")`, then `statusRow(col, 'Looks good', …)` for each item with nothing to suggest — phrased positively and specifically ("Text contrast checked on all 14 text layers").
3. `statusRow(col, 'Not on this screen', …)` for checklist items with no matching UI.
4. `subhead(col, "For dev & QA")`, then `statusRow(col, 'Confirm in build', "SC 2.1.1 Keyboard — WCAG_2.1.1-Keyboard. Check every control responds to Enter/Space.")` for the related items this section carries.

Card copy follows the voice rules in SKILL.md: the suggested value as the title, no failure language, one decision per element.

## Step G — The "For dev & QA" section (Run everything only)

`createCheckSection(page, groupBy('devqa'), master)` with **no** screenshot copy and no callouts — remove the cloned `img` right after creation, since these items aren't tied to one element. Then a single column of `statusRow(col, 'Confirm in build', …)` rows for every item not carried by another section (reflow, zoom, text spacing, orientation…), each with its WCAG SC and JIRA tag and a one-line note on how to check it.

## Step H — Fit, name, restack, and rebuild the overview

Run at the end of every section build (and once more at the very end of a multi-check run).

```js
function fitSection(section, pad = 64) {
  let maxX = 0, maxY = 0;
  for (const c of section.children) {
    maxX = Math.max(maxX, c.x + c.width);
    maxY = Math.max(maxY, c.y + c.height);
  }
  section.resizeWithoutConstraints(maxX + pad, maxY + pad);
}

// Counts live in the section name, because setPluginData isn't available.
// e.g. "1 · Accessible names — 7 suggestions"   "4 · Focus order — 12 tab stops"
function nameSection(section, g, count) {
  const unit = g.key === 'focus' ? 'tab stop' : g.key === 'devqa' ? 'note' : 'suggestion';
  section.name = `${g.n} · ${g.title} — ${count} ${unit}${count === 1 ? '' : 's'}`;
}

// Keep sections in group order under the overview, whatever order they were run in.
function restack(page) {
  const overview = page.findOne(n => n.name === 'Start here');
  let y = overview ? overview.y + overview.height + STACK_GAP : 0;
  const sections = checkSections(page).sort((a, b) => parseInt(a.name) - parseInt(b.name));
  for (const s of sections) { s.x = 0; s.y = y; y += s.height + STACK_GAP; }
}
```

### The "Start here" overview

A page-level auto-layout frame at the top. It replaces the old pass/fail scorecard: the designer is just starting their annotations, so it shows **what's been suggested and what's left to look at**, not a score. **No pass counts, no fail counts, no "X / 30".**

```js
function buildOverview(page, frameName, isImageInput) {
  const old = page.findOne(n => n.name === 'Start here');
  if (old) old.remove();

  const ov = figma.createAutoLayout('VERTICAL'); ov.name = 'Start here';
  page.appendChild(ov);
  ov.x = 0; ov.y = 0;
  ov.itemSpacing = 12;
  ov.paddingLeft = ov.paddingRight = ov.paddingTop = ov.paddingBottom = 32;
  ov.cornerRadius = 12;
  ov.fills = [{ type: "SOLID", color: WHITE }];
  ov.strokes = [{ type: "SOLID", color: { r: 0.86, g: 0.86, b: 0.86 } }];
  ov.resize(960, ov.height);
  ov.layoutSizingHorizontal = 'FIXED';

  addText(ov, 'Accessibility head start', 32, "Bold");
  addText(ov, frameName, 16, "Semi Bold", MUTED);
  addText(ov, 'A first pass at Blueline annotations and design suggestions for this screen. ' +
    'Each suggestion is a starting point: review it, adjust it where your product context says otherwise, ' +
    'then add it in Stark.', 14, "Regular");
  if (isImageInput) {
    addText(ov, 'Made from a screenshot, so sizes and colors are estimated from pixels.', 13, "Regular", MUTED);
  }

  const done = Object.fromEntries(checkSections(page).map(s => [parseInt(s.name), s.name.split(' — ')[1] || '']));
  for (const g of GROUPS) {
    if (g.key === 'devqa' && !(g.n in done)) continue;   // only listed when it was built
    const row = figma.createAutoLayout('HORIZONTAL'); row.name = `Row: ${g.title}`;
    row.itemSpacing = 10;
    row.counterAxisAlignItems = 'CENTER';
    row.fills = [];
    ov.appendChild(row);
    row.layoutSizingHorizontal = 'FILL';
    numberPill(row, g.n, g.color);
    const label = addText(row, g.title, 14, "Semi Bold");
    label.layoutSizingHorizontal = 'FIXED'; label.resize(200, label.height);
    addText(row, g.n in done ? done[g.n] : 'Not run yet', 14, "Regular", g.n in done ? INK : MUTED);
  }

  addText(ov, 'Badges in each section are numbered to match the cards beside them. ' +
    'In Focus order, the number is the tab stop.', MIN_TEXT, "Regular", MUTED);
  return ov;
}
```

Order of calls at the end of each run: `fitSection(section)` → `nameSection(section, g, count)` → `buildOverview(page, …)` → `restack(page)`.

## Verifying the result

**Screenshot the section node (or the page), not the screenshot copy.** `get_screenshot` on a node renders only that node's subtree; callouts are siblings of the screenshot copy inside the section, so a screenshot of the copy alone shows no callouts. That's easy to misread as "the markers didn't get created".

Take one screenshot per finished section and check, in order:
1. **No badge sits on the screenshot copy.** Check the corners: badges from adjacent lanes shouldn't touch, and second-ring badges shouldn't run into the suggestion column.
2. **Every leader line touches the element its card describes** — not a neighbour, not empty space. A line pointing at the wrong thing is worse than no callout.
3. **No two badges collide**, and lines don't cross awkwardly where it's easy to avoid.
4. **Badge numbers match card numbers**, and every card reads suggestion → why → details.
5. **Nothing reads as a verdict.** Scan the card titles: each should be a suggested value ("Accessible name: 'Print'"), with no "missing", "fails" or "issue".
6. **No text is clipped or under 12px**, and the section label and overview row match.

As a hard check, re-run `get_metadata` on the *original* frame's node id and diff it against the very first read. It must be identical. If a callout looks wrong, fix it with a targeted `use_figma` call (adjust that item's `bx`/`by` and redraw its callout, or move the one badge) rather than rebuilding the section.

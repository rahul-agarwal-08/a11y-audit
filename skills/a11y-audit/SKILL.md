---
name: a11y-audit
description: Gives designers a head start on accessibility annotations for a UI design — from a Figma file/frame link OR a plain UI screenshot — by suggesting Blueline annotations (accessible names, heading levels, landmarks, focus order) and design tweaks (contrast, touch targets, link text, form labels) based on Adobe's Design Accessibility Checklist (WCAG 2.2 AA). Lets the user pick which checks to run, run everything, or go one check at a time, then writes each check into Figma as its own clearly labelled section on a new review page. Use this whenever the user shares a Figma link or UI screenshot and asks for an accessibility review, a11y audit, WCAG check, Blueline annotations, Stark prep, or "is this design accessible" — even if they don't say "accessibility" (e.g. "can you check this screen", "annotate this for engineering handoff", "help me with bluelines", "what should the heading levels be"). Never modifies the original frame, file, or page.
---

# a11y-audit

## What this skill is (and isn't)

This skill is a **head start**, not a grade. The designer using it is usually at the very beginning of working out what their Blueline annotations should be. Your job is to do the first pass for them: look at the design, decide what each annotation should most likely be, and write those suggestions into Figma so they can review, adjust and carry them into Stark.

So the output is a set of **suggestions with reasons**, organised by check, not a list of failures. There is no pass/fail score. Nothing in the output should read as "you missed this" or "this is wrong" — the design is unfinished by definition, and the designer asked for help, not a verdict.

**The original frame is read-only for the entire run.** Every write happens on a page or file that didn't exist before the run started. A Blueline annotation is a record for Engineering, not an edit to the design, and this skill keeps that principle absolutely.

Reference files — read them as you reach the step that needs them:
- `references/adobe_checklist.md` — every checklist item with its WCAG SC, JIRA tag and Blueline type, plus which check group each item belongs to.
- `references/bluelines_conventions.md` — the 4 Blueline annotation types, naming conventions, and Stark's known false positives.
- `references/wcag_and_accname.md` — accessible-name computation, tab-vs-arrow-key logic, and how to make the judgment calls.
- `references/figma_write_recipes.md` — the Plugin API patterns for Step 5. Read it before your first `use_figma` call.

## Step 1 — Identify the input and load prerequisites

The user will give you either:
- **A Figma URL** — may point to a specific frame (`?node-id=...`) or a whole file.
- **A local image path** — a screenshot of a UI, with no Figma source.

Before touching any Figma tool, load its prerequisite skill — skipping this causes silent, hard-to-debug failures:
- Before any `use_figma` call → load `figma-use`.
- Before any `get_design_context` call → load `figma-design-to-code`.
- Before any `create_new_file` call → load `figma-create-new-file`.

Figma's live-state tools (`use_figma`, `get_metadata`, `get_screenshot`) act on the Figma **desktop app's** current state. If a file isn't already open there (including a brand-new file you just created), ask the user to open it before you proceed.

**Figma input with no node-id and more than one frame:** ask whether to cover just one frame or every frame. A whole flow is a very different deliverable from one screen.

## Step 2 — Let the user choose what to run

Running every check is the slowest path, because each check adds analysis and Figma write calls. So before doing the heavy work, offer the choice. **Skip this step if the user's message already says what they want** ("just do headings", "full audit", "everything") — then go straight to that.

Use the environment's structured question tool if one is available (e.g. `AskUserQuestion`, `ask_user_input`); otherwise ask in chat as a short numbered list. Keep it to one round of questions.

**Question 1 — How would you like to run it?**
1. **One check at a time** (recommended) — start with Accessible names, review it, then choose the next.
2. **Pick specific checks**
3. **Run everything** — all checks in one go; noticeably slower.

**If they pick specific checks**, ask a multi-select follow-up. If the question tool caps options at 4, split into two questions:
- *Blueline annotations:* Accessible names · Headings · Landmarks · Focus order
- *Design suggestions:* Color & contrast · Touch targets · Text & links · Forms & errors

### The check groups

Each group becomes **one section** on the review page. Run them in this order when running more than one; in one-at-a-time mode, suggest them in this order.

| # | Group | Blueline type | Checklist items | Dev & QA notes it carries |
|---|---|---|---|---|
| 1 | Accessible names | Accessible name | 1, 2, 3 | — |
| 2 | Headings | Heading | 8 | — |
| 3 | Landmarks | Landmark | 23, 24 | 25, 26 (if one screen) |
| 4 | Focus order | Focus order | 17, 18 | 15, 16 |
| 5 | Color & contrast | Design suggestion | 4, 5, 6, 7 | — |
| 6 | Touch targets | Design suggestion | 22, 19 | 20, 21 |
| 7 | Text & links | Design suggestion | 10, plus Stark typography (text under 12px, full justification) | 9 |
| 8 | Forms & errors | Design suggestion | 12, 11, 13, 14 (static parts) | 11, 13, 14 (behaviour) |
| 9 | For dev & QA | — | — | 27, 28, 29, 30, and any notes not already carried above — **only in "Run everything"** |

Groups 1–4 produce **Blueline annotations**: the designer is expected to add these, so every relevant element gets a suggestion. Groups 5–8 produce **design suggestions**: optional tweaks that would make the design easier to use, each with a priority.

"Dev & QA notes" are checklist items a static design can't show (keyboard behaviour, motion timing, zoom/reflow). They go at the bottom of the section they relate to, framed as things worth confirming in the build — never silently dropped.

## Step 3 — Build the element inventory (once per conversation)

Build it once and reuse it for every check in this conversation, including later one-at-a-time runs. Re-fetching metadata for each check is the single biggest avoidable slowdown.

**Figma input:** call `get_metadata` on the frame for the structural tree (every child's id, type, name, position, size), and `get_design_context` for text content, fills/colors and a screenshot. These are exact, not estimated. Call `get_design_context` with `disableCodeConnect: true` — the audit doesn't need Code Connect, and without the flag the tool may return a Code Connect setup prompt instead of design data.

**Drill in only where the chosen checks need it.** `get_metadata` does not expand component instances, so icon buttons nested inside instances (toolbars, prompt boxes, card headers) are invisible to it. When a chosen check depends on them — especially Focus order or Accessible names — run **one read-only `use_figma` script** that loads the frame's page (`await page.loadAsync()`), walks `frame.findAll(...)`, and returns each relevant node's `absoluteBoundingBox` relative to the frame (and `absoluteRenderBounds` for text). It must not mutate anything; see "Read-only measurement" in `references/figma_write_recipes.md`. In testing, guessing a toolbar's internals from the screenshot attributed one component's icon cluster to a different component sitting next to it. If only Headings or Color & contrast were chosen, don't drill into icon toolbars at all.

**Image input:** call `whoami` for the plan key, then `create_new_file` (editorType `design`, name like `"<original name> — Accessibility head start"`). Ask the user to open it in the desktop app, then `upload_assets` (no nodeId) to place the image. Read the image file directly too, so you can inspect icon meaning, legibility and color usage. Note once, prominently, in the overview frame that sizes and colors are **estimated from pixels**.

End with a flat list of every meaningful node: id/reference, type (icon button, image, text, input, link, nav group…), bounding box, visible text, and relevant style info.

## Step 4 — Work through each chosen check

For each checklist item in the chosen groups, classify it:
- **Suggestion** — something to annotate (groups 1–4) or a tweak worth making (groups 5–8). Write a suggestion card.
- **Looks good** — checked, nothing to suggest. One short line, phrased positively ("Text contrast checked on all 14 text layers").
- **Not on this screen** — no matching UI pattern (e.g. no form inputs). One short line.
- **For dev & QA** — can't be confirmed from a static design. One short line with its WCAG SC and JIRA tag, under the related section.

### Make the call — one suggestion per element

You're doing the first pass so the designer doesn't have to. For every element, **commit to one concrete suggestion**: the actual name, level, region, tab position or color value. Don't list alternatives, don't write "consider X or Y", and don't stop to ask about individual elements.

When you're unsure, still pick the most likely answer, set confidence to Medium or Low, and add one clause saying what would change it ("if this icon opens settings rather than filters, use 'Settings'"). The only reason to pause and ask is a structural call that would reshape a whole section (e.g. whether an entire content block is a landmark at all).

Use the judgment logic in `references/wcag_and_accname.md`: decorative vs. meaningful ("would removing it lose information or a way to operate the UI?"), heading vs. styled text ("does it introduce the content after it?"), and tab-stop vs. arrow-key group ("does the group have one 'current item'?"). For Focus order, propose a full sequence from reading order (header → primary nav → content → secondary chrome), treating true composite widgets (tabs, toolbars, trees, radio groups) as one tab stop with arrow keys inside.

### Each suggestion has three parts, in this order

1. **The suggestion** — the value itself, written as the card title, plus the element it applies to.
   - `Accessible name: "Print"` — printer icon button, top-right toolbar
   - `Heading level: H2` — "Map collections" text
   - `Landmark: Navigation, named "Product navigation"` — left rail
   - `Tab stop 4: Search field`
   - `Text color: #464646 (5.1:1 on white)` — helper text under the title field
   - `Hit area: 24 × 24px` — close icon in the toast
2. **Why it helps** — one or two sentences on who benefits and how. "Screen reader users hear this name when they reach the button, and voice-control users say it to press it."
3. **Details** — Blueline type (or "Design suggestion"), WCAG SC + level, JIRA tag, confidence, and — for design suggestions only — priority.

### Voice: a head start, not a grade

- **Describe the design, not the designer.** Never "you missed", "you forgot", "you should have", "it's wrong".
- **Don't use failure language** in titles or explanations: no "fail", "issue", "violation", "problem", "error" (except when talking about a UI's own error messages), "missing", "lacks", "non-compliant".
- **Use direction words:** "Suggested", "Use", "Add", "Set", "Mark as", "Name it".
- **State facts neutrally.** "At 3.2:1, this helper text sits under the 4.5:1 that text this size needs" — not "contrast fails".
- **Lead with the answer.** The first thing anyone reads on a card is the suggested value.

| Instead of | Write |
|---|---|
| Missing accessible name on icon button | Accessible name: "Filter" |
| Heading level skipped (H1 → H3) | Heading level: H2 |
| Contrast fails AA (3.2:1) | Text color: #5C5C5C (4.6:1 on white) |
| Touch target too small | Hit area: 24 × 24px |
| You used placeholder text as a label | Add a visible label: "Email address" |

### Priority (design suggestions only)

- **High** — without it, some people can't complete the task: an input with no label at all, meaning carried by color alone at a decision point, a primary action that can't be reached.
- **Medium** — makes the task noticeably harder: text contrast under AA, a primary-action hit area under 24px, link text with no context.
- **Low** — polish and best practice: text under 12px (no WCAG minimum), full justification, an optional skip link.

Blueline sections (1–4) carry no priority: every annotation there is one the designer will add anyway.

## Step 5 — Write each check into Figma as its own section

Follow `references/figma_write_recipes.md`. The shape of the output:

1. **One review page** (Figma input) or the new file's page (image input), named `Accessibility head start — <frame name>`. Never the original page. Later runs in the same conversation add to this same page.
2. **A "Start here" overview** at the top of the page: the title, one line on what this is ("A first pass at Blueline annotations and design suggestions — review, adjust, then add them in Stark"), and one row per section showing its name and count ("Accessible names · 7 suggestions"). It also lists checks not run yet, so one-at-a-time users can see what's left. Rebuild it after every run so counts stay current. **No pass/fail score and no "X / 30".**
3. **One Figma Section per check group**, stacked below the overview, labelled `<#> · <Group name>` with a one-line description under the label. Each section holds:
   - its own copy of the flattened reference screenshot,
   - numbered callout badges (1, 2, 3… restarting in every section; in Focus order the number *is* the tab stop) in the section's color, with leader lines to the exact element, sitting in the margin — never on top of the UI,
   - a column of suggestion cards (suggestion → why → details), numbered to match the badges,
   - "Looks good" and "Not on this screen" lines,
   - a "For dev & QA" block for the related items that need a build to confirm.
4. **A "For dev & QA" section** at the end, only in "Run everything", for the items not carried by any other section.

Each section has its own color, but color is never the only cue: every section is also spatially separate, labelled, and its badges are numbered to match its cards.

**Hard safety constraint, non-negotiable:** every `use_figma` call must first find-or-create the review page and switch to it, and must only ever mutate nodes created by this skill. Node ids from `get_metadata`/`get_design_context` on the original frame are read-only inputs — they may appear in text you write, never as the target of `resize`, `x =`, `fills =`, `characters =`, `remove()` or any other mutating call.

### Keep it fast

- One inventory and one reference screenshot per conversation; every section clones the screenshot rather than re-uploading it.
- Build each section in as few `use_figma` calls as possible. A whole section (screenshot copy, callouts, cards, overview refresh) fits in a single call, and several sections can be batched into one call when the user picked more than one check.
- Take one verification screenshot per section, after it's complete, not after every call.

## Step 6 — Tell the user what happened

Keep the chat summary short; the Figma page is the deliverable. Say where the page lives (link if you have one), what was added ("Accessible names section: 7 suggestions, 2 looks-good checks"), and highlight two or three of the most useful suggestions in the same suggestion-first voice.

**In one-at-a-time mode**, end by offering the next check with the question tool: the next two or three groups in table order, plus "That's enough for now". When they pick one, reuse the inventory and add its section to the same page.

# Adobe Design Accessibility Checklist (WCAG 2.2 AA)

This is the reference rubric for `a11y-audit`. Each item: **requirement — WCAG SC — JIRA tag — Blueline type (or Design suggestion)**. "Blueline type" means this is one of the 4 official Stark annotation types (Accessible name, Heading, Landmark, Focus order); "Design suggestion" means it's a checklist item Stark doesn't have a dedicated annotation for — still cover it, just don't present it as a Blueline annotation.

The skill uses these items to decide **what to suggest**, not to score the design. There's no pass/fail total in the output.

## Check groups (what the user picks from, and how sections are organised)

| # | Group | Items | Dev & QA notes carried |
|---|---|---|---|
| 1 | Accessible names | 1, 2, 3 | — |
| 2 | Headings | 8 | — |
| 3 | Landmarks | 23, 24 | 25, 26 (only checkable across screens) |
| 4 | Focus order | 17, 18 | 15, 16 |
| 5 | Color & contrast | 4, 5, 6, 7 | — |
| 6 | Touch targets | 22, 19 | 20, 21 |
| 7 | Text & links | 10, plus Stark typography (text under 12px, full justification) | 9 |
| 8 | Forms & errors | 12, and the static parts of 11, 13, 14 | 11, 13, 14 (behaviour) |
| 9 | For dev & QA (Run everything only) | — | 27, 28, 29, 30, plus anything not carried above |

## 1. Accessible names
1. Specify alternate text for informative images — **SC 1.1.1 Non-text Content (A)** — `WCAG_1.1.1-Non-text_Content` — **Accessible name**
2. Specify when graphics are decorative and to be ignored by AT — **SC 1.1.1 (A)** — `WCAG_1.1.1-Non-text_Content` — **Accessible name**
3. Specify accessible names for controls (icon-only buttons, functional images) — **SC 4.1.2 Name, Role, Value (A)** — `WCAG_4.1.2-Name_Role_Value` — **Accessible name**

## 2. Color
4. Do not convey meaning with color alone — **SC 1.4.1 Use of Color (A)** — `WCAG_1.4.1-Use_of_Color` — Design suggestion
5. Regular text ≥4.5:1 contrast to background — **SC 1.4.3 Contrast Minimum (AA)** — `WCAG_1.4.3-Contrast_Minimum` — Design suggestion
6. Large text (≥24px, or ≥18.5px bold) ≥3:1 contrast — **SC 1.4.3 (AA)** — `WCAG_1.4.3-Contrast_Minimum` — Design suggestion
7. UI elements/graphical objects ≥3:1 contrast to background — **SC 1.4.11 Non-text Contrast (AA)** — `WCAG_1.4.11-Non-text_Contrast` — Design suggestion

## 3. Content and motion
8. Headings create structure in the UI (real h1–h6 markup, not just styled text) — **SC 1.3.1 Info and Relationships (A)** — `WCAG_1.3.1-Info_and_Relationships` — **Heading**
9. No movement over 5 seconds without a means to stop it — **SC 2.2.2 Pause, Stop, Hide (A)** — `WCAG_2.2.2-Pause_Stop_Hide` — Design suggestion — *usually "Cannot verify" for a static design unless motion/prototype is present*
10. Write meaningful link text (no "click here"/"learn more" with no context) — **SC 2.4.4 Link Purpose In Context (A)** — `WCAG_2.4.4-Link_Purpose` — Design suggestion

## 4. Forms and errors
11. Provide explicit error messaging (what's wrong + how to fix) — **SC 3.3.1 Error Identification (A)** — `WCAG_3.3.1-Error_Identification` — Design suggestion
12. All inputs have real labels, not placeholder-as-label — **SC 3.3.2 Labels or Instructions (A)** — `WCAG_3.3.2-Labels_or_Instructions` — Design suggestion
13. When the user makes an input error, give suggestions for valid input — **SC 3.3.3 Error Suggestion (AA)** — `WCAG_3.3.3-Error_Suggestion` — Design suggestion
14. Provide a review/correct opportunity before irreversible consequential actions (legal/financial/data) — **SC 3.3.4 Error Prevention (AA)** — `WCAG_3.3.3-Error_Suggestion` (Adobe's own checklist reuses this tag) — Design suggestion

## 5. Keyboard
15. All mouse functionality available via keyboard — **SC 2.1.1 Keyboard (A)** — `WCAG_2.1.1-Keyboard` — Design suggestion — *"Cannot verify" from a static design — whether a control mechanically responds to keyboard input is a code fact, not a layout fact*
16. Specify character key shortcuts (single-key shortcuts must be remappable) — **SC 2.1.4 Character Key Shortcuts (A)** — `WCAG_2.1.4-Character_Key_Shortcuts` — Design suggestion — *"Cannot verify" statically — remappability is a code/settings fact*
17. Specify keyboard focus order — **SC 2.4.3 Focus Order (A)** — `WCAG_2.4.3-Focus_Order` — **Focus order** — **Propose, don't defer.** Unlike 15/16, this one *should* get a concrete suggested sequence from the audit, the same way accessible names and heading levels get concrete suggestions — infer it from visual/reading order (top-to-bottom, left-to-right within regions; header → primary nav → content → secondary chrome is a reasonable default) and Adobe's own grouping convention: a cohesive control cluster (a tab strip, a toolbar, a tree) is usually ONE tab stop with a note that arrow keys move within it, rather than one tab stop per sub-control. Flag genuinely ambiguous groupings (e.g. "is this a button group or an arrow-key tab set?") with Medium/Low confidence instead of guessing silently.
18. Provide strong/visible focus indicators — **SC 2.4.7 Focus Visible (AA)** — `WCAG_2.4.7-Focus_Visible` — Design suggestion — *checkable only if a focused-state variant is shown in the file*

## 6. Touch and gesture
19. All interactions achievable via single touch/click (no path-based/multipoint-only gestures) — **SC 2.5.1 Pointer Gestures (A)** — `WCAG_2.5.1-Pointer_Gestures` — Design suggestion
20. Provide alternate methods to physical inputs like shaking/tilting, with an off switch — **SC 2.5.4 Motion Actuation (A)** — `WCAG_2.5.4-Motion_Actuation` — Design suggestion — *usually "Cannot verify" statically*
21. Provide alternatives to dragging gestures **[new in WCAG 2.2]** — **SC 2.5.7 Dragging Movements (AA)** — no Adobe JIRA tag confirmed as of Aug 2024 per source doc — Design suggestion
22. Touch targets have a 24×24px minimum hit area **[new in WCAG 2.2]** — **SC 2.5.8 Target Size Minimum (AA)** — no Adobe JIRA tag confirmed as of Aug 2024 per source doc — Design suggestion — *directly measurable from bounding boxes, high-confidence category*

## 7. Navigation
23. Assign UI a landmark region (not a hard WCAG requirement, but encouraged; if used, ALL content must be inside some landmark) — **SC 2.4.1 Bypass Blocks (A)** — `WCAG_2.4.1-Bypass_Blocks` — **Landmark**
24. Skip-to-main-content link (optional per WCAG, standardized as Adobe best practice) — **SC 2.4.1 (A)** — `WCAG_2.4.1-Bypass_Blocks` — Design suggestion (best practice, not required)
25. Nav controls in the same location/order throughout the experience — **SC 3.2.3 Consistent Navigation (AA)** — `WCAG_3.2.3-Consistent_Navigation` — Design suggestion — *only checkable with multiple screens*
26. UI components identified consistently across the whole experience — **SC 3.2.4 Consistent Identification (AA)** — `WCAG_3.2.3-Consistent_Navigation` (Adobe's own checklist reuses this tag) — Design suggestion — *only checkable with multiple screens*

## 8. Reflow and zoom
27. Designs work in portrait and landscape (unless orientation is genuinely essential) — **SC 1.3.4 Orientation (AA)** — `WCAG_1.3.4-Orientation` — Design suggestion — *"Cannot verify" unless multiple orientation variants are provided*
28. Support 200% zoom / 640px effective viewport — **SC 1.4.4 Resize Text (AA)** — `WCAG_1.4.4-Resize_text` — Design suggestion — *"Cannot verify" from a single static frame*
29. Support 400% zoom / 320px effective viewport — **SC 1.4.10 Reflow (AA)** — `WCAG_1.4.10-Reflow` — Design suggestion — *"Cannot verify" from a single static frame*
30. Users can independently adjust text spacing in-browser (line height ≥1.5×, paragraph spacing ≥2×, letter spacing ≥0.12×, word spacing ≥0.16× font size) — **SC 1.4.12 Text Spacing (AA)**, web only — `WCAG_1.4.12-Text_Spacing` — Design suggestion — *always "Cannot verify" statically*

*(Note: Adobe's source checklist groups items 27–30 as 4 items under "Reflow and zoom" but its own progress counter totals 29 — items 13/14 and 25/26 share JIRA tags in the source, which is why the raw item count is slightly higher than 29; report against all listed items regardless of the exact total, and don't force the count to reconcile to exactly 29 if your inventory finds more or fewer applicable items for a given design. The skill doesn't report a score, so this count never appears in the output.)*

## Quick lookup: which items can be suggested from a static design vs. confirmed only in the build

**Reliably verifiable/proposable from a still design/screenshot:** 1, 2, 3, 4, 5, 6, 7, 8, 10, 12, 17 (propose a sequence, per above), 22, 23, 24.
**Verifiable only with extra context (multiple screens/states provided):** 18, 25, 26, 27.
**Always a "For dev & QA" note for a single static frame:** 9, 11, 13, 14, 15, 16, 19, 20, 21, 28, 29, 30.

## Notable example patterns worth pattern-matching against

- Never rely on border color alone for form errors — pair with icon + helper text.
- Contrast worked example: palette blue 600 heading text over `#f7f7f7` = 2.44:1 → fails large-text 3:1 minimum.
- Quiet-variant links inside paragraphs need underline (not just color) to reliably signal "this is a link," even if contrast alone clears 3:1.
- Two `nav` landmarks on one page must get distinct accessible names (e.g. "Site navigation" vs. a second nav) or screen reader users can't tell them apart.
- A component's accessible name (e.g. "App switcher") must stay identical everywhere it appears, even across merged/stitched product surfaces.

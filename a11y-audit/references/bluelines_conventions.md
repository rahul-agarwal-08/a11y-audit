# Blueline / Stark annotation conventions

Adobe designers annotate Figma files with 4 Blueline annotation types via the Stark plugin, for Engineering handoff. `a11y-audit` mirrors these conventions in its own output so the report reads as a native extension of a workflow designers already know — but note this skill produces its own marker badges via Figma Plugin API code, not literal Stark plugin output.

## The 4 annotation types

1. **Accessible name** — for elements with no visible text label (icon-only buttons, informative images). Adobe explicitly does **not** use "Alt text" as a type — it's not platform-agnostic (iOS uses `.accessibilityLabel`, Android `contentDescription`, HTML `aria-label`/`alt`, desktop `AutomationProperties.Name` — all are "accessible name," none should be called "alt text" in the annotation). When writing a suggested name, keep it 1-3 words and function-focused ("Print", not "Blue printer icon").
2. **Heading** — applied to text layers only, choosing a level h1–h6. Never skip levels. Styling (bold, large) is not sufficient evidence of heading intent — look for the content role (does this text introduce/group content that follows it?).
3. **Landmark** — applied to top-level frames/regions (Header/Banner, Navigation, Main, Region/Section, Aside, Footer/ContentInfo). A landmark needs an explicit accessible name when: (a) more than one of the same type exists on a screen, or (b) it's a Region/Section landmark (always needs a name). Don't over-mark — not every visual block should be a landmark, and landmarks shouldn't be used inside modal dialogs.
4. **Focus order** — a sequence of numbers documenting tab order across interactive elements, with notes for arrow-key navigation within a group, Escape-to-close behavior, or keyboard shortcuts.

## Stark's automated test tools (context, not literally invoked by this skill)

- **Contrast**: 4.5:1 regular text, 3:1 large text (≥24px or ≥18.5px bold)/UI elements.
- **Typography**: flags <12px text (safe to ignore — no WCAG minimum) and full justification (must-fix, Adobe/Spectrum requirement not a WCAG one).
- **Touch targets**: flags <24px hit areas; 44px flagged separately as optional AAA.
- **Vision simulator**: color vision deficiency simulation (start with Achromatopsia/grayscale to catch color-only meaning fastest).

## Known false positives designers are told to ignore (useful calibration for this skill's own judgment)

- An element with a proper Accessible name annotation will still get flagged "missing alt text" by Stark's automated scanner, because the scanner only checks for the (unused) Alt text field, not the Accessible name annotation. **Don't repeat this false positive in this skill's own suggestions** — if an element clearly has a sensible name in context, don't flag it just because there's no literal `alt` attribute.
- Text below 12px is a best-practice nudge, not a WCAG requirement — suggest it as Low priority at most.
- A component-library file with no page-level landmarks is expected and fine — don't flag missing landmarks on files that are clearly component libraries rather than product screens (many small variants on one page, no page-level chrome).

## Why this skill separates "Blueline annotations" from "Design suggestions"

Only 4 of the checklist items correspond to something a designer would literally annotate with Stark in Figma. The rest (contrast, touch targets, color-alone, forms/errors, navigation consistency, reflow/zoom) are real WCAG requirements Adobe still tracks, just without a dedicated Stark marker type. Keeping them in separate sections prevents implying that, say, a contrast tweak is something to annotate in Stark — it's a change to the actual color value instead. Blueline sections (names, headings, landmarks, focus order) hold annotations the designer will add; design-suggestion sections hold optional tweaks, each with a priority.

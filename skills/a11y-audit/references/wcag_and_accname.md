# WCAG structure and accessible-name computation (grounding reference)

Use this to reason about *why* a suggestion helps, and to make naming, heading and focus-order calls consistently.

## POUR principles and conformance levels

WCAG 2.2 has 87 success criteria under 4 principles:
- **Perceivable** — content must be presentable in ways users can perceive (alt text, captions, contrast).
- **Operable** — UI must be operable (keyboard access, no seizure-inducing content, enough time).
- **Understandable** — content/operation must be understandable (readable, predictable, input assistance).
- **Robust** — content must work with current/future assistive tech (valid markup, correct name/role/value — this is where accessible names live).

Levels are cumulative: **A** (32 criteria, minimum baseline) → **AA** (+24, the real-world legal/practical standard — this is the target Adobe's checklist tests against) → **AAA** (+31, enhanced/specialized, rarely mandated site-wide).

## Accessible Name computation (AccName) — precedence order

When judging what an element's accessible name *should* be, or whether a design's stated name is being computed the way the designer intends, use this precedence (highest wins):

1. `aria-labelledby` (or platform equivalent referencing another element's text)
2. `aria-label` (or platform equivalent: iOS `.accessibilityLabel`, Android `contentDescription`, desktop `AutomationProperties.Name`)
3. Native host-language labeling (HTML `<label>`, `alt`, SVG `<desc>`)
4. The element's own visible text content
5. `title` attribute, as a last resort (should not be relied on — not consistently exposed by all AT)

This is why Adobe's checklist prefers connecting a visible label to a control (SC-agnostic best practice) over an invisible-only name: a visible label benefits sighted users, voice-control users (who need to say what they see), AND screen reader users simultaneously, while an invisible-only `aria-label` only serves the last group.

## Tab vs. arrow keys — deciding a group's keyboard pattern (for Focus Order proposals)

When proposing a Focus Order sequence (SC 2.4.3), the recurring judgment call is: does this cluster of controls get **one Tab stop with arrow-key navigation inside it** (a WAI-ARIA APG "composite widget"), or does **each control get its own separate Tab stop**? Deciding wrong is an easy, common mistake — don't infer it from visual grouping alone.

**The deciding question: does the group have an internal "current item" concept?** Radio buttons have one checked value; tabs have one active tab; a tree has one focused node; a grid has one active cell. If yes, it's a composite widget — arrow keys move the internal current item, Tab skips the whole group in one stop. If the elements are just independent actions sitting near each other with no shared "current selection," each gets its own Tab stop — that's the default, and it's what plain HTML gives you for free.

**APG-defined composite widgets (arrow keys + one Tab stop):** Menu/Menubar, Listbox, Radio Group, Tabs, Toolbar, Tree View/Treegrid, Grid (data table), Combobox popup.

**Commonly mis-assumed as composite but usually NOT (each item gets its own Tab stop):** Breadcrumbs (standard implementation is a plain `nav > ol > li > a` list — each link is an independent navigation action, there's no "current breadcrumb item" concept). A row of visually-grouped but functionally-independent buttons (e.g. Save / Delete / Share) — visual grouping into a toolbar-*looking* row does not by itself make it the APG Toolbar pattern; that requires an actual design decision to group them for tabbing efficiency.

**A third, easily-confused case: arrow keys that change a *value*, not focus.** Sliders, spinbuttons/steppers, 2D color pickers — one Tab stop, and the arrow key adjusts the control's value rather than moving to a different element. Don't count "arrow keys are involved" as evidence of a composite widget — check whether focus is moving between elements or a value is changing on the one focused element.

**Practical rule for this audit:** default to separate Tab stops unless the group clearly matches one of the APG composite patterns above. When a real screen has an ambiguous case (a row of icon buttons that *could* be either pattern), say so explicitly with a lower confidence rating rather than picking one silently — the visual shape never disambiguates this, only product/engineering intent does.

## How this shapes judgment calls in the audit

- **Naming an icon-only button**: if a universally-recognized icon (print, close, search, settings) — High confidence suggestion. If a custom/ambiguous icon with no adjacent text — Low confidence, and say so explicitly rather than inventing a confident-sounding name.
- **Heading vs. styled text**: ask "does this text introduce or group the content that follows it in the layout?" not "is this text big/bold?" A pull-quote or callout box may be large/bold without being a heading at all.
- **Decorative vs. meaningful image**: ask "if this were removed, would a user lose information or a way to operate the interface?" If yes → meaningful, needs a name. If no (pure texture, redundant icon next to already-labeled text, spacer) → decorative, flag for Engineering to hide from the accessibility tree.
- **Landmark boundaries**: landmarks should map to genuine page regions (header/nav/main/footer), not every visual card or section. When two of the same landmark type coexist (e.g. two `nav` regions), both need distinguishing accessible names.

# Changelog

## 1.0.0 — 2026-10-09

First public release.

- Choose what to run: one check at a time, specific checks, or everything.
- One Figma Section per check (Accessible names, Headings, Landmarks, Focus order, Color & contrast, Touch targets, Text & links, Forms & errors, For dev & QA) under a "Start here" overview. No pass/fail score.
- Suggestion-first cards (suggestion → why it helps → details) and a voice guide with no failure language.
- Numbered callouts with leader lines in the margin; lanes confined to the image edges, second-ring overflow and an overlap pass.
- Reference screenshot exported inside Figma (`exportAsync` + `createImage`), with `upload_assets` as fallback.
- Read-only measurement of nodes inside component instances; text anchored from `absoluteRenderBounds`.
- Anchors on the element edge facing its badge; preset lanes respected.
- `get_design_context` called with Code Connect disabled.
- Fixed `createAutoLayout` naming (the options argument is ignored by the API).
- Palette and text sizes checked against WCAG: all text ≥ 12px, all text/fill pairs ≥ 4.5:1.

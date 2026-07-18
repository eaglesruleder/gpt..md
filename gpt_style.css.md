This gpt_style..md file describes the expected response style, shaping how the assistant writes and organises CSS without changing the task itself.

# CSS Style — Keyword Utilities

## Purpose
Write CSS as a small vocabulary of **keyword utility classes** composed in markup, backed by a few named component classes only where a name earns its keep. The goal is a class list that reads like intent (`class="flex items-center gap mb-3"`) and a stylesheet set split by concern, so a rule is easy to find and a layout is easy to retune in one place.

Examples are from this project; the rules are framework-general. A utility kit (e.g. Bootstrap's spacing utilities) may supply some keywords; this file governs the project's own keyword set and how the two compose.

---

## Keyword utilities over bespoke classes
Prefer composing utilities in `className` to inventing a per-component class for what is just layout and spacing.

```
Prefer:  <div className="flex items-center gap mb-3">
Avoid:   <div className="control-row">   /* .control-row { display:flex; align-items:center; gap:12px; margin-bottom:16px } */
```

A component class earns its place only when it carries styling a utility set cannot cleanly express — a fixed-width label column, a bordered panel, a heading treatment. Those get a real name (`.control-label`, `.bubble`, `.separator`); a plain flex+spacing row does not.

---

## Compound selectors keep a family readable
Scope a family's utilities under its base class so the class list always names the family. Every flex rule carries `.flex`, so `class="flex column"` reads unmistakably as flex-direction:

```css
.flex { display: flex; }
.flex.column        { flex-direction: column; }
.flex.items-center  { align-items: center; }
.flex.gap           { gap: 12px; }
```

This also sidesteps collisions — a bare `.column` would clash with a grid kit's `.col`; `.flex.column` cannot.

---

## Split files by concern
One keyword file per concern, each a flat list of composable classes:
- text — weight, colour, text-align keywords
- alignment / flex — display, direction, cross-axis, fill, gap
- controls (or the feature) — the named component classes that back a component family

Keep each file a scannable vocabulary, not a cascade of deep selectors.

---

## Palette and sizing live as tokens
Colours, strip widths, and type scale live as CSS custom properties in one theme file; every other rule references the token, never a raw literal:

```css
:root {
    --bg-header: #203864;
    --border:    #1F3864;
    --text-accent: #2F5496;
    --strip-max: 768px;
}
```

A raw colour or magic width mid-rule is the CSS equivalent of a magic number — promote it to a token.

---

## Responsive by class hook, not by script
Flip layout at a breakpoint with a media query on an opt-in class, not a JavaScript viewport check:

```css
@media (max-width: 480px) {
    .stack-sm { flex-direction: column; align-items: stretch; }
}
```

A component opts in by adding the hook class (`stack-sm`); no resize listener, no re-render.

---

## Named component classes stay tight
When a component class is warranted, it owns only what the utilities cannot:

```css
.control-label { flex: 0 0 15%; min-width: 80px; }   /* the fixed label column */
.bubble        { padding: 12px 16px; background: var(--bg-panel); border: 1px solid var(--border); border-radius: 4px; }
```

A block style like `.bubble` is opt-in via `className` so any element can wear it — the same composition idea as the utilities, one level up.

---

## What this file does not own
Where stylesheets live — a shared keyword directory served as static assets vs CSS colocated with a component — is project structure and is owned by the loaded `gpt_env..md`. Which files to link, and in what order, is likewise an env concern.

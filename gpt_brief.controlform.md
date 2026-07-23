This gpt_brief..md file defines the **ControlForm object model** — the JSON a form author (human or GPT) fills in to build a form for the OddWire.Form runtime. It is a build-a-form reference, not a task document.

# ControlForm — Build-a-Form Reference

## Purpose
Give another GPT everything it needs to author a working form as JSON, without reading the runtime source. A form is a **JSON control tree**; a live document is a **sparse `param`-keyed overlay** over that tree; the runtime dispatches each control by `type`. Radio/dropdown options may resolve from a lookup database.

Runtime specifics (structure, persistence, hazards) live in `gpt_env.oddform.md`. This file owns the object schema.

---

## The form object

```ts
FormDefinition = {
    formId: string;                  // GUID; minted on save if absent
    label?: string;                  // form title (masthead + landing list)
    version?: string;                // shown next to the label; drives install/update status
    export?: FormExportDefinition;   // enables the header export actions (see Export)
    displayParam?: (string|null)[];  // params projected into an instance's landing-list label; null = detail-line break
    groupParam?: string | string[];  // params the landing list groups instances by
    filterParam?: string[];          // params the landing list builds filter dropdowns from
    orderParam?: string | string[];  // params the landing list can order instances by
    dateModified?: string;           // ISO; author-provided (forms are "published"), not stamped
    controls: ControlDef[];          // the control tree
    }
```

A form is loaded from static JSON at startup and seeded into a local store. Author `formId` as a stable GUID. `displayParam`/`groupParam`/`filterParam`/`orderParam` drive only the landing-page list views; omit them for a form that is never listed.

---

## Common props — every control (`ControlDefBase`)

| Prop | Type | Meaning |
|---|---|---|
| `type` | string | the discriminant — selects the control (see catalogue) |
| `param` | string | **identity**; the value is stored under this key. Flat across the whole form (incl. nested layout children) — must be unique |
| `label?` | string | field label |
| `value?` | (per type) | default/seed value |
| `hidden?` | boolean | omit from render |
| `disabled?` | boolean | render read-but-not-editable (greyed) |
| `placeholder?` | string | empty-state hint (dropdown empty option, text input) |
| `stacked?` | boolean | label above the field instead of beside it |
| `rows?` | number | textarea height in rows (textarea only; defaults to 3) |
| `cellClassName?` | string | grid-cell span classes for column layout (see Columns) |
| `pdf?` | map | page → print boxes for PDF export (see PDF placement) |

`readonly` and `className` exist on the runtime control props but are **not JSON-authorable** — they are used programmatically (e.g. the DB Manager), not in form definitions.

### Columns (`cellClassName`)
Every control list is a 12-column grid; a control is **full-width unless** it sets `cellClassName`. Spans: `col-1`…`col-12`; responsive `sm-col-N` (≥480px) and `lg-col-N` (≥768px) override from that breakpoint up. Author responsive multi-column rows by making cells add to 12:

```json
{ "type": "text", "param": "name", "label": "Name",      "stacked": true, "cellClassName": "sm-col-12 lg-col-8" },
{ "type": "text", "param": "cr",   "label": "Challenge",  "stacked": true, "cellClassName": "sm-col-12 lg-col-4" }
```
→ full-width stacked on phones, `name` (8) + `cr` (4) side-by-side on wide screens.

---

## Control catalogue (the `ControlDef` union)

### Leaf controls

| `type` | `value` | Extra props | Renders |
|---|---|---|---|
| `label` | string | `labelFor?` | static text (heading + value); no input |
| `text` | string | `valueType?`, `keyboardType?` | single-line input |
| `textarea` | string | `rows?` | multi-line input (auto-stacked label) |
| `checkbox` | boolean | — | single checkbox |
| `image` | string | — | static image; `value` is an image URL loaded into `<img>` (capture/upload is future) |
| `radio` | string | `controls?`, `dbOptions?` | inline radio group |
| `dropdown` | string | `controls?`, `dbOptions?` | `<select>` |

- `valueType`: `'text' | 'int' | 'decimal' | 'email' | 'phone'` — drives the mobile keyboard; **stored as the raw string** (no parse/coerce).
- `keyboardType`: overrides the inferred `inputMode`.
- `controls` on radio/dropdown is the **static option list**: `ControlOption = { value: string; label: string }` — `value` is saved, `label` is shown.
- A `label` control renders its `label` prop as a **heading** (a separator line) and its `value` as **body text** beneath. Both are optional: label-only is a heading/divider, value-only is a plain text line, both together give heading + body. It never takes input, so use value-only `label`s to lay out static list lines.
- `labelFor` on a `label` mirrors another control's `hidden` state — the label hides when the named control is hidden.

### Layout controls
Structural; own local UI state (open/active), store **nothing** — `param` is identity only. They render a nested control list over the **same flat instance**.

| `type` | Extra props | Renders |
|---|---|---|
| `collapsible` | `controls: ControlDef[]`, `subtitle?` | titled expand/collapse panel; `subtitle` is a smaller line under the title |
| `tab` | `controls: ControlDef[]` | one tab section; **adjacent `tab`s group into one tabset** |
| `popup` | `controls: ControlDef[]` | button that opens a modal of the children |
| `looper` | `controls: ControlDef[]`, `addRows?` | repeating group — renders `controls` once per row; `addRows` shows a "+ Add Row" button |

- `controls` here is **child controls**, not options.
- A **run of adjacent `tab` controls** becomes one tabset. If `controls[0].type === 'tab'`, the whole form renders as a **root bottom-bar tabset** (page navigation).
- A `hidden` tab is dropped from its tabset.
- A `looper` stores a **row-instance array** under its `param` (`value: LooperRowInstance[]`), each row a sparse overlay over the looper's `controls`. Unlike other layout controls, its child `param`s are scoped **per row**, not to the flat form. See **Authoring a `looper`** below for the row JSON.
- A `looper` renders **no heading of its own** — precede it with a `label` control (or wrap it in a `collapsible`) to title the section.

### Authoring a `looper`
A `looper`'s `value` is an **array of row instances**. Each row is itself a mini form-instance over the looper's `controls` — the **same sparse `param`-keyed overlay** shape as a whole `FormInstance`, **not** a flat `{ child: value }` map:

```json
{
    "type": "looper", "param": "traits", "addRows": true,
    "controls": [
        { "type": "text",     "param": "note", "cellClassName": "sm-col-11" },
        { "type": "checkbox", "param": "done", "cellClassName": "sm-col-1" }
    ],
    "value": [
        { "controls": [ { "param": "note", "value": "First row text" } ] },
        { "controls": [ { "param": "note", "value": "Second row text" } ] }
    ]
}
```

- **Row shape is `{ "controls": [ { "param", "value" }, … ] }`** — the wrapping `controls` array is mandatory. A flat `{ "note": "…" }` will **not** seed the row.
- Only the child `param`s you want to seed need appear; the rest fall back to the child control defs (e.g. the `checkbox` above defaults unchecked, so its `done` param is omitted per row).
- A bare `{}` — or any row missing its `controls` array — normalises to **one empty row**. That is the idiom for a single blank starter row: `"value": [ {} ]`.
- `addRows: true` shows the **+ Add Row** button; omit it for a fixed row set.
- Child `param`s are scoped **per row**, so the same child `param` (`note`, `done`, …) repeats down every row — and across other loopers — without colliding with the flat form scope. Only the looper's **own** `param` must be unique in the form.
- The `{ text · checkbox }` split above (`sm-col-11` / `sm-col-1`) is the canonical checklist row: an editable line with a tick beside it. Every leaf honours `cellClassName`, so a looper row lays out on the 12-col grid like any other control list.

---

## Lookup-backed options — `dbOptions`

Radio/dropdown can source options from a lookup table instead of (or alongside) static `controls`:

```ts
dbOptions = string | {
    table: string;                 // lookup table name
    valueParam: string;            // row column saved as the value (the KEY)
    labelParam?: string;           // row column displayed (defaults to valueParam)
    filter?: string | { formParam: string; tableParam: string };
    joinOptions?: boolean;         // true → static `controls` + db options
    fill?: boolean;                // on select, write EVERY column of the chosen row into its matching param
    }
```

- **String form** (`"Model"`) — bare table name; value & label default to the table schema's first column.
- **`filter`** — dependent/cascading list: keep rows where `row[tableParam] === instance[formParam]`. A string means `formParam === tableParam`. No upstream value → no options.
- **Critical rule:** the control **saves `valueParam` (the key), displays `labelParam`** — never save the label.
- **`fill`** — a "record" pick: selecting a row also writes each of its columns into the instance param of the same name (e.g. a monster dropdown fills `name`/`size`/`ac`/… from the chosen row). Requires the table's column names to match the form's control `param`s.
- Missing table or zero (post-filter) options → the control renders **disabled with a "No Options"** placeholder.

The lookup database is edited in the Settings DB Manager, keyed by scope → table name (`'_global'` shared; per-form scope is future). A lookup table:

```ts
LookupTable = {
    tableName: string;
    lastUpdated?: string;
    schema: ControlDef[];              // column defs (same ControlDef shape; simple types)
    rows: Record<string, unknown>[];   // param→value records
    }
```

---

## Export — API + PDF

A form opts into the header export actions with an `export` block:

```ts
FormExportDefinition = {
    api?: string | boolean | { url?: string };   // POST the flattened instance as JSON to this URL
    pdf?: boolean | { enabled?: boolean };        // enable client-side PDF export
    url?: string;                                 // fallback endpoint for api when api is a boolean
    }
```

- **Flattening** resolves the instance to a `param→value` record (hidden controls included; lookup controls export the stored key). API export posts this JSON; PDF export draws it.
- **Value templating (export-only):** a control `value` may embed `{param}` tokens, interpolated against the flattened values at export time — e.g. a hidden `{ "param": "sizeType", "value": "{size} {type}", "hidden": true }` emits one combined field. Same token syntax as label templating, but resolved into the exported value rather than the on-screen label. The form UI does not render these.

### PDF placement (`pdf`)
Any control can carry a `pdf` map so its value prints at fixed page coordinates onto a stored template (PDF, or PNG/JPEG background):

```ts
pdf = Record<string, ControlPdfBox[]>;   // page-index key ("0") → one or more boxes

ControlPdfBox = {
    x: number; y: number;                       // bottom-left anchor (pdf-lib origin: page bottom-left)
    w?: number;                                 // max width + enables word-wrap; absent → collapses to the x/y anchor
    h?: number;                                 // block bound: ellipsis-clip, or the shrink target
    fontSize?: number;                          // per-box override of the app default size
    align?:  'left' | 'center' | 'right';       // within [x, x+w] when w set, else about the anchor
    valign?: 'top' | 'middle' | 'bottom';       // within [y, y+h] when h set, else about the anchor
    shrinkToFit?: boolean;                       // step the font down until the text fits w (and h)
    }
```

- A `checkbox`'s box prints `X` when true, blank when false.
- An `image` control's box draws the image (aspect-fit, centered) instead of text; needs both `w` and `h`.
- The template document is supplied per form (bundled package or import), not in the form JSON.

---

## The live document — instances (sparse overlay)

You rarely author instances by hand, but the shape matters:

```ts
FormInstance = {
    formId?: string;
    instanceId?: string;               // GUID; minted on save
    dateModified?: string;             // stamped by the store
    controls: { param: string; value?: unknown; [override: string]: unknown }[];
    }
```

- Each entry patches the matching form control **by `param`**; instance props override form props (sparse — only changed params appear).
- Duplicate `param`s collapse **last-wins**.
- Edits emit `onChange(value, param)`; the runtime patches the instance immutably.

---

## Rules & gotchas (author these correctly)

- **`param` is the whole identity.** Flat scope — a nested layout child reusing a top-level `param` silently shares its value. Keep every `param` unique across the entire form.
- **Save the key, show the label** for any lookup control (`valueParam`, not `labelParam`).
- **Unknown `type`** renders a visible error box, never crashes — so a typo in `type` fails loud but safe.
- **`displayParam`** picks which params form the instance's label in the landing list; changing it re-projects existing instances.
- **State is memory-first, persisted locally** (no server). Nothing you author is validated at load beyond the type switch — a malformed control surfaces only at render.
- **Layout children share the root `onChange`** — no scoping; a collapsible/tab/popup is pure presentation. The one exception is `looper`, whose children are scoped per row (see catalogue).
- **Label templating:** any `label` may embed `{param}` tokens, interpolated against the instance's current values at render — e.g. a tab `"label": "STR {strSave}"` renders `STR +3` once `strSave` is filled, and updates live. Works on leaf and tab labels.

---

## Minimal example

```json
{
    "formId": "11111111-1111-4111-8111-111111111111",
    "label": "Contact",
    "displayParam": ["fullName"],
    "controls": [
        { "type": "label", "param": "title", "label": "Contact" },
        { "type": "text", "param": "fullName", "label": "Full name" },
        { "type": "text", "param": "email", "label": "Email", "valueType": "email" },
        { "type": "checkbox", "param": "subscribed", "label": "Subscribe", "value": true }
    ]
}
```

## Lookup + cascade example

```json
{
    "formId": "22222222-2222-4222-8222-222222222222",
    "label": "Vehicle",
    "displayParam": ["model", "stock"],
    "controls": [
        { "type": "dropdown", "param": "model", "label": "Model",
          "dbOptions": { "table": "Model", "valueParam": "id", "labelParam": "name" } },
        { "type": "dropdown", "param": "stock", "label": "On lot",
          "dbOptions": { "table": "OnLot", "valueParam": "stock", "labelParam": "rego", "filter": "model" } }
    ]
}
```

## Tabbed page + nested layout example

```json
{
    "formId": "33333333-3333-4333-8333-333333333333",
    "label": "Job",
    "controls": [
        { "type": "tab", "param": "tabDetails", "label": "Details", "controls": [
            { "type": "text", "param": "ref", "label": "Reference" },
            { "type": "collapsible", "param": "moreSection", "label": "More", "controls": [
                { "type": "textarea", "param": "notes", "label": "Notes" }
            ]}
        ]},
        { "type": "tab", "param": "tabContact", "label": "Contact", "controls": [
            { "type": "text", "param": "phone", "label": "Phone", "valueType": "phone" }
        ]}
    ]
}
```

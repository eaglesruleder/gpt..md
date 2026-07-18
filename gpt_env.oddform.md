This gpt_env..md file describes a working environment including its platforms, structures, defaults, and common gotchas so other gpt..md prompts can reason inside the right local context.

# OddWire.Form Environment

## Purpose
Define the local operating environment for the OddWire.Form dynamic form runtime — an ASP.NET Core host serving a React/TypeScript single-page app that renders JSON-driven forms.

Focus:
- what the product is for
- solution + client/server layout
- client source structure
- the runtime's data model (form definition, instance overlay, lookup database)
- state ownership and where it persists
- stack, tooling, and dev-server gotchas
- the failure points directly tied to that structure

This file describes the environment and its shape, not the build order or any one feature. It should stay true across releases.

---

## Product Objective

OddWire.Form is a **config-driven form runtime**: a form is authored once as JSON — a tree of typed controls — and the app renders it, captures a user's answers as a live document, and can export that document. The intent is that a non-developer (or a GPT) can define and evolve a form by editing data, never the renderer.

The product's three standing goals:
- **Author by data** — a new form or control is a JSON edit, not a code change; the runtime dispatches every control by `type`.
- **Edit as a sparse overlay** — a filled-in document stores only what differs from the definition, keyed by `param`.
- **Own its data locally, export outward** — everything lives browser-local; export paths (API POST, PDF) push a resolved document out to other systems.

Correctness is judged by observable in-browser behaviour, not by compile/type success alone.

---

## Core Environment Model

OddWire.Form is **client-first, host-thin**.

Default assumptions:
- the dynamic form runtime lives entirely in the React client
- the ASP.NET Core server is template scaffold — a host and dev-time proxy, not a form API
- forms, instances, and lookup data seed from static local JSON and then persist **browser-local** (via `localforage`/IndexedDB)
- there is no backend store or cross-device sync; "persisted" means on this browser
- compile/type success is not enough; the observable behaviour is what matters

Think in this order:
1. solution root
2. client vs server
3. client source area (components / context+stores / pages / export / settings / data)
4. the runtime data model (definition → instance → dispatch)
5. lookup database (for radio/dropdown options)
6. in-browser behaviour

---

## Project Structure

### 1. Solution root
- `OddWire.Form.slnx` — solution file
- `OddWire.Form.Server/` — ASP.NET Core host
- `oddwire.form.client/` — React/TypeScript client (the product)

### 2. Server — `OddWire.Form.Server`
ASP.NET Core, standard SPA-host template. Assume:
- it exists to host the client and proxy in development
- it carries only scaffold endpoints (e.g. the template `WeatherForecast` controller); no form/instance/lookup endpoints exist
- an export path may POST to a server endpoint through the dev proxy, but the server does not own form data

### 3. Client — `oddwire.form.client`
React 19 + TypeScript + Vite, routed with `react-router-dom`. UI via `react-bootstrap` + `bootstrap`. Code follows `gpt_style.pseudocode.md` (general), `gpt_style.react.md` (TSX surface), and `gpt_style.css.md` (keyword CSS). Source layout under `src/`:
- `_components/` — reusable presentational components, one leaf folder per family with an `index.ts` barrel
  - `controllist/` — the JSON renderer: iterate a control scope, resolve each control against the instance overlay, dispatch by `type`; holds the `controls/` leaf+layout controls (over a shared `ControlBase` label+field wrapper), the `lookup/` option resolver, and the shared control-type contracts in `controls/controlTypes`
  - `layout/` — page shells (e.g. the strip masthead)
- `_context/` — the store singletons and their React contexts, the live-document entity, shared types, and seed/static JSON under `data/`
- `form/` — the active-form route/page that composes the renderer and owns the save flow
- `landing/` — the form/instance picker
- `settings/` — settings screens (lookup DB management, bundled-form install)
- `export/` — resolved-instance flattening and the export paths (API, PDF)
- `mods/` — bundled optional data importers
- `App.tsx` / `main.tsx` — router + app root; `AppSettings.json` — app-level config
- `public/style/` — global keyword stylesheets linked from `index.html` (`theme`, `layout`, `text`, `alignment`, `controls`)

**CSS location rule:** component-immediate CSS is colocated next to its component (`Component.css`, imported by it); shared/global CSS is a concern-split keyword file in `public/style/` linked from `index.html`. Which files to link, and in what order, is set in `index.html`.

---

## Stack & Tooling

- React 19, TypeScript, Vite (dev + build); `npm run dev`, `npm run build` (build runs `tsc -b` then `vite build`)
- Routing: `react-router-dom` (`BrowserRouter`)
- Local persistence: `localforage` (IndexedDB) under one database, `oddwire.form`, split into per-concern stores
- Export: `pdf-lib` (client-side PDF), `jszip` (bundled form packages)
- UI: `react-bootstrap` + `bootstrap`. KendoReact is intentionally deferred to data-heavy screens (its licence/weight is not worth it for simple controls)
- `resolveJsonModule` is enabled — form/instance/lookup JSON is imported directly
- Dev server runs **HTTPS** on port `59392` (override `DEV_SERVER_PORT`) with the ASP.NET dev certificate, and proxies `/api` and the scaffold `/weatherforecast` to the server
- Preview/browser tools may reject the self-signed cert until `dotnet dev-certs https --trust` is run once

---

## Runtime Data Model

The central structural rule:

**A form is a JSON control tree. A live document is a sparse array of `param`-keyed patches over that tree. The runtime dispatches each control by `type`.**

- **Form definition** — `{ formId, label?, controls: ControlDef[] }`; each control has `type`, `param`, and type-specific props.
- **Form instance** — `{ formId?, instanceId?, controls: ControlInstance[] }`; each patch is `{ param, ...overrides }`. Sparse; matched to form controls by `param`; instance props override form props.
- **Resolution** — the renderer merges `{ ...formControl, ...instanceControl }`, then switches on the resolved `type`. Unknown type → safe fallback, never a crash.
- **Editing** — leaf controls emit `onChange(value, param, subkey?)`; the live-document entity patches the instance array immutably and the page re-renders.
- **Layout controls** — nested/structural controls render a nested control list over the **same flat instance** and the **same root onChange**; they own only local UI state, no value.
- **Lookup** — radio/dropdown with a `dbOptions` config resolve options from the lookup database: **save the key, display the label**; missing table → fallback.

---

## State & Persistence Ownership

State is **browser-local**: seeded from static JSON on first run, then persisted through `localforage` (IndexedDB) — it survives a refresh; it does **not** sync to a server or across devices.

- store singletons own the persisted data — forms + form index, instances + instance index, the lookup database, and export template blobs — each in its own `localforage` store under the `oddwire.form` database; a context provider initialises them (forms before instances, since instance display projection reads the form index) and supplies them to the tree
- the active page owns the current form and a **live-document entity** in React state; the entity applies edits and owns debounced self-persistence
- a new instance is **in-memory only until the first explicit save**; after that, edits autosave (debounced), and unmount/navigation flushes any pending write
- the lookup database is held in its store and mutated by the settings DB management screens
- layout UI state (expanded, active tab, modal open) and editor drafts are local and ephemeral

Server-side storage, cross-device sync, and multi-user concerns are outside this environment.

---

## Conventions & Defaults

- code follows `gpt_style.pseudocode.md` (general applied-pseudocode), `gpt_style.react.md` (TSX surface: 4-space, brace-on-own-line, leading-comma, arrow-vs-function, prop order, `{...props}` layout wrappers, barrels), and `gpt_style.css.md` (keyword utility classes); regions are rare in TSX and stay unused while files are small
- shared control contracts live in `_components/controllist/controls/controlTypes`; keep them small and extend the discriminated `ControlDef` union when adding a control type
- persisted data is owned by store singletons (one `localforage` store per concern), provided through React context; pages hold view state and the live-document entity, not the stores
- untyped JSON is cast once at the load/merge boundary (`as unknown as`), never scattered — this is where real runtime validation would later attach
- scope is **flat**: `param` is the identity across the whole instance, including nested layout children
- navigation is `react-router-dom` routes off a `formId` / optional `instanceId` (and equivalent settings/export routes)

---

## Common Structural Risk Checks

Check these early:
- duplicate `param` within a scope — controls share one instance entry and collide `key={param}`; instance duplicates resolve last-wins
- flat scope means a layout child reusing a top-level `param` silently shares its value
- a lookup control saves the key, not the label; a saved key whose row was deleted or renamed shows blank — the missing fallback only fires for a whole missing table, not a missing row
- persistence is browser-local: state survives refresh but is confined to this browser and store; clearing site data or switching browser/device loses it, and there is no server of record
- JSON boundary casts bypass the type system — malformed definitions surface only at runtime, not at load
- HTTPS dev cert rejected by the browser/preview until trusted

Prefer these distinctions:
- definition issue vs instance issue vs lookup issue
- runtime (client) issue vs host (server) scaffold — most bugs are client-side
- persisted-state issue vs in-memory view-state issue
- type/compile success vs observed in-browser behaviour

---

## Working Summary

When in doubt, assume OddWire.Form is structured like this:
- `OddWire.Form.slnx` with a thin ASP.NET host and a React/TS client
- the form runtime is entirely client-side, routed with `react-router-dom`
- forms/instances/lookup seed from static JSON, then persist browser-local via `localforage`
- a form is a JSON control tree; a document is a sparse `param`-keyed instance overlay
- the renderer dispatches by `type`; unknown types fall back safely
- radio/dropdown resolve options from the lookup database and store the key
- state is browser-local (survives refresh, no server); export paths push a resolved document outward
- correctness is judged by in-browser behaviour, not compilation alone

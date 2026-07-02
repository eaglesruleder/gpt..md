This gpt_env..md file describes a working environment including its platforms, structures, defaults, and common gotchas so other gpt..md prompts can reason inside the right local context.

# OddWire.Form Environment

## Purpose
Define the local operating environment for the OddWire.Form dynamic form runtime — an ASP.NET Core host serving a React/TypeScript single-page app that renders JSON-driven forms.

Focus:
- solution + client/server layout
- client source structure
- the runtime's data model (form definition, instance overlay, lookup database)
- state ownership and where it does and does not persist
- stack, tooling, and dev-server gotchas
- the failure points directly tied to that structure

---

## Core Environment Model

OddWire.Form is **client-first, host-thin**.

Default assumptions:
- the dynamic form runtime lives entirely in the React client
- the ASP.NET Core server is template scaffold — a host and dev-time proxy, not a form API
- forms, instances, and lookup data are loaded from static local JSON into memory at startup
- all runtime state is **memory-only**; there is no persistence, no backend writes, no autosave
- compile/type success is not enough; the observable behaviour is what matters

Think in this order:
1. solution root
2. client vs server
3. client source area (runtime / context / pages / settings / data)
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
- it carries only scaffold endpoints (e.g. the template `WeatherForecast` controller); **no form/instance/lookup endpoints exist**
- adding real persistence would start here, but that is out of current scope

### 3. Client — `oddwire.form.client`
React 19 + TypeScript + Vite. UI via `react-bootstrap` + `bootstrap`. Source layout:
- `src/form/` — the runtime
  - `ControlList` (iterate), `ControlItem` (resolve + dispatch by type)
  - `controls/` leaf controls; `controls/layout/` collapsible/tab/popup
  - `types/` — `controlTypes`, `instanceTypes`, `lookupTypes`
  - `utils/instanceMerge`, `lookup/LookupResolver`
  - `dynamicform.git.md` / `dynamicform.gpt.md` — feature brief + implementation map
- `src/context/` — `AppContext`, `AppInitializer` (loads JSON, holds mutable lookup state), `AppContextActions` (pure DB transforms)
- `src/pages/` — `LandingPage`, `FormPage`, `SettingsPage`
- `src/settings/` — `DbManager` + `Db*`, `ImportFromFileButton`
- `src/data/` — `forms/`, `instances/`, `lookup/` static JSON

---

## Stack & Tooling

- React 19, TypeScript, Vite (dev + build); `npm run dev`, `npm run build` (build runs `tsc -b` then `vite build`)
- UI: `react-bootstrap` + `bootstrap`. KendoReact is intentionally deferred to data-heavy screens (its licence/weight is not worth it for simple controls)
- `resolveJsonModule` is enabled — form/instance/lookup JSON is imported directly
- Dev server runs **HTTPS** on port `59392` with the ASP.NET dev certificate and proxies API calls to the server
- Preview/browser tools may reject the self-signed cert until `dotnet dev-certs https --trust` is run once

---

## Runtime Data Model

The central structural rule:

**A form is a JSON control tree. A live document is a sparse array of `param`-keyed patches over that tree. The runtime dispatches each control by `type`.**

- **Form definition** — `{ formId, label?, controls: ControlDef[] }`; each control has `type`, `param`, and type-specific props.
- **Form instance** — `{ formId?, instanceId?, controls: ControlInstance[] }`; each patch is `{ param, ...overrides }`. Sparse; matched to form controls by `param`; instance props override form props.
- **Resolution** — `ControlItem` merges `{ ...formControl, ...instanceControl }`, then switches on the resolved `type`. Unknown type → safe fallback, never a crash.
- **Editing** — leaf controls emit `onChange(value, param)`; `FormPage` patches the instance array immutably and re-renders.
- **Layout controls** — collapsible/tab/popup render a nested `ControlList` over the **same flat instance** and the **same root onChange**.
- **Lookup** — radio/dropdown with a `lookup` config resolve options from the lookup database: **save the key, display the label**; missing table → fallback.

---

## State & Persistence Ownership

Assume **memory-only** throughout:
- `FormPage` owns the active instance in local state
- `AppInitializer` holds the mutable lookup database in state; DB Manager edits mutate it
- layout UI state (expanded, active tab, modal open) and DB editor drafts are local and ephemeral
- nothing survives a refresh; navigation away from a form discards unsaved instance edits

Persistence, autosave, and server reads/writes are explicitly future work, not current behaviour.

---

## Conventions & Defaults

- code follows `gpt_style.pseudocode.md`; region markers use the TS/JS form `//#region` / `//#endregion`, and only where a body needs translating
- shared contracts live in `src/form/types/`; keep them small and add to the discriminated `ControlDef` union when adding a control type
- context mutations are pure functions in `AppContextActions`, bound to state in `AppInitializer`
- untyped JSON is cast once at the load/merge boundary (`as unknown as`), never scattered — this is where real runtime validation would later attach
- scope is **flat**: `param` is the identity across the whole instance, including nested layout children
- navigation is a simple `Screen` state union in `App` (no router)

---

## Common Structural Risk Checks

Check these early:
- duplicate `param` within a scope — controls share one instance entry and collide `key={param}`; instance duplicates resolve last-wins
- flat scope means a layout child reusing a top-level `param` silently shares its value
- a lookup control's saved key whose row was deleted/renamed shows blank with no fallback (the `missing` fallback only fires for a missing whole table)
- importing into an existing table keeps the old schema but replaces rows — column mismatch hides fields
- editing a schema column's `param` orphans existing row data keyed by the old name
- memory-only state: any edit is lost on refresh or navigation; make this explicit in UI or debug output
- JSON boundary casts bypass the type system — malformed definitions surface only at runtime
- HTTPS dev cert rejected by the browser/preview until trusted

Prefer these distinctions:
- definition issue vs instance issue vs lookup issue
- runtime (client) issue vs host (server) scaffold — most bugs are client-side
- type/compile success vs observed in-browser behaviour

---

## Roadmap Context

Delivered: MVP stages 1–4 (hard controls → JSON `ControlList`/`ControlItem` → `FormPage` instance state → local context + navigation) and both MVP+ addons (layout controls; DB Manager / lookup data).

Not yet built (out of current scope): server/database persistence and autosave, validation and rules engines, looper/repeating controls, image/file controls, CSV import, and unique-`param` validation.

---

## Working Summary

When in doubt, assume OddWire.Form is structured like this:
- `OddWire.Form.slnx` with a thin ASP.NET host and a React/TS client
- the form runtime is entirely client-side
- forms/instances/lookup load from static JSON into memory
- a form is a JSON control tree; a document is a sparse `param`-keyed instance overlay
- `ControlItem` dispatches by `type`; unknown types fall back safely
- radio/dropdown resolve options from the lookup database and store the key
- all state is memory-only; persistence and validation are future work
- correctness is judged by in-browser behaviour, not compilation alone

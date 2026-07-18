This gpt_style..md file describes the expected response style, shaping how the assistant writes, structures, and presents React/TypeScript output without changing the task itself.

# React / TSX Style

## Purpose
Write React + TypeScript that reads like the applied-pseudocode philosophy in `gpt_style.pseudocode.md`, in the concrete surface conventions this project uses. This file owns the React/TSX **surface**: file shape, component shape, prop handling, and JSX layout. It does not restate the general vertical-layout and comment discipline — that lives in `gpt_style.pseudocode.md` §6–8 and §11 and is referenced here.

Examples are TS/React; the rules are React-general. Swap `react-bootstrap` for another component kit and the rules still hold.

---

## Relationship to gpt_style.pseudocode.md
`gpt_style.pseudocode.md` owns the language-general layer:
- vertical operator-led boolean/arithmetic chains, one term per line
- comma-led argument lists and initializers, closing delimiter on its own line
- `// Intent:` / `// Objective:` load-bearing comments; untagged restating comments deleted
- region markers are language-relative — in TSX, folding happens at component and JSX-element boundaries, so method-local regions are rare

This file adds only what is specific to writing React components. Where the two overlap (leading-comma initializers), the pseudocode file is the master; apply it here unchanged.

---

## Formatting

- **Indent 4 spaces.**
- **Opening brace on its own line** for function declarations:

```tsx
export function FormPage()
{
    ...
}
```

- **Single quotes** for strings and imports.
- **Named exports**, not default exports — a leaf folder's `index.ts` barrel re-exports them.
- **Blank line between third-party imports and local imports:**

```tsx
import Form from 'react-bootstrap/Form';

import { ControlBase } from './ControlBase';
import type { CoreControlProps } from './controlTypes';
```

- **Multi-line arrays and objects use leading-comma vertical layout** (per `gpt_style.pseudocode.md` §7):

```tsx
const keyboardForValueType: Record<TextValueType, KeyboardType> =
    {text: 'text'
    ,int: 'numeric'
    ,decimal: 'decimal'
    };
```

- **Type literals** keep `& {` inline with `;`-separated members and the closing `};` indented one level:

```tsx
type ControlDropdownProps = CoreControlProps<string> & {
    controls?: ControlOption[];
    };
```

---

## Component shape

**Arrow component with an implicit return** when the body is a single JSX expression:

```tsx
export const ControlDropdown = (props: ControlDropdownProps) =>
    <ControlBase {...props}>
        <Form.Select value={props.value ?? ''} onChange={e => props.onChange?.(e.target.value, props.param)}>
            ...
        </Form.Select>
    </ControlBase>;
```

**`function` declaration (brace on its own line)** when the body needs an early return or a local:

```tsx
export function ControlCheckbox(props: CoreControlProps<boolean>)
{
    if (props.hidden)
        return null;

    return (
        <div className="flex items-center gap mb-3">
            ...
        </div>
        );
}
```

The closing `);` of a returned JSX block aligns with the JSX root element, not the `return`.

---

## Props

**Prefer a whole-`props` parameter and spread it into a layout wrapper**, so adding a wrapper prop does not touch every leaf:

```tsx
export const ControlTextField = (props: ControlTextFieldProps) =>
    <ControlBase {...props}>
        <Form.Control ... value={props.value ?? ''} onChange={e => props.onChange?.(e.target.value, props.param)} />
    </ControlBase>;
```

Destructure instead only when the component does not forward to a wrapper (a self-contained leaf).

**A layout wrapper takes the layout slice of the shared props type, not a hand-redeclared one:**

```tsx
type ControlBaseProps = Pick<CoreControlProps<unknown>, 'param' | 'label' | 'hidden' | 'stacked'> & {
    className?: string;
    children: ReactNode;
    };
```

`Pick` (or `Omit`) keeps the wrapper honest — it declares only what it lays out, and the leaf's extra props (`value`, `onChange`, `valueType`, `controls`) ride along the spread as ignored extras.

**Delegate cross-cutting gates to the wrapper.** A `hidden` gate, error display, or required marker lives once in the base, so leaves that use it do not repeat `if (hidden) return null`. A leaf that does not use the base gates itself.

**Prop order inside a JSX element** is consistent so elements scan the same way:
1. keys — `key`, `id`, `name`
2. component typing — `type`, `as`
3. `label`
4. value — `value`, `checked`
5. value typing — `inputMode`, and similar
6. functions — `onChange` and other handlers
7. style — `className`, `inline`, `rows`

```tsx
<Form.Check
    key={option.value}
    id={`${props.param}-${option.value}`}
    name={props.param}
    type="radio"
    label={option.label}
    checked={props.value === option.value}
    onChange={() => props.onChange?.(option.value, props.param)}
    inline
/>
```

---

## Barrel exports
Each leaf folder (e.g. `controls/`, `layout/`) carries an `index.ts` that re-exports its components, so a consumer imports several from one path:

```ts
export { ControlText } from './ControlText';
export { ControlTextField } from './ControlTextField';
```

```tsx
import { ControlText, ControlTextField, ControlRadio } from '../_components/controls';
```

An internal-only helper a folder's components share (e.g. a base wrapper) need not be re-exported.

---

## Comments
Follow `gpt_style.pseudocode.md` §8: no comments that restate the code. Keep only load-bearing `// Intent:` / `// Objective:` notes. Explanatory prose that narrates what a component does is deleted — the component name and prop shape carry it.

---

## What styling this file does not own
Class names, CSS keyword vocabulary, and the CSS file layout are owned by `gpt_style.css.md`. Where component CSS files live (component-colocated vs a shared keyword directory) is project structure and is owned by the loaded `gpt_env..md`.

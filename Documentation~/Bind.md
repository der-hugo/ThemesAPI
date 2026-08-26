Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - Bind Components</h1>

This page documents the binding components and their inspector: how a themed value is selected per selectable state, the value/modifier UX, and how targets and transitions are configured.

## Contents

- [Overview](#overview)
- [Binding Modes](#binding-modes)
- [Selectable States](#selectable-states)
- [Selecting a Value](#selecting-a-value)
- [Add a Value from the Binding](#add-a-value-from-the-binding)
- [Value Modifiers](#value-modifiers)
- [Apply To (Target)](#apply-to-target)
- [With Animation (Fade Duration)](#with-animation-fade-duration)
- [Rename Safety](#rename-safety)
- [Related](#related)

---

## Overview

A binding is a `Bind<TValue, TTarget>` component: it resolves a themed `TValue` from the active [`ThemeDatabase`](ThemeDatabase.md) and applies it to a `TTarget` (a component or GameObject). Use a [built-in binding](ScriptingAPI.md#built-in-bindings) (for example `BindColorToGraphic`, `BindSpriteToImage`, `BindStringToTextMeshPro`) or a [custom binding](ScriptingAPI.md#custom-binding-examples).

The values a binding stores are plain, rename-safe [`ThemeValue<T>`](ScriptingAPI.md#themevalue-and-value-types) references - each holds the selected definition GUID and an optional local modifier.

[![][1]][1]

## Binding Modes

The **Type** dropdown (shown when the binding is driven by a bindable selectable) selects how values are resolved:

- **Static** - always applies the single value, ignoring selectable state.
- **Selectable** - applies the value for the current selectable state (see [Selectable States](#selectable-states)).
- **Toggle** - like `Selectable`, but when the driving [`BindableToggle`](ScriptingAPI.md#bindable-components) is **off** it uses the separate `Off Values` block. Only available when the source is a toggle.

[![][8]][8]

**Listen To** shows the bindable selectable that drives the binding. It is discovered automatically from the hierarchy (a bindable ancestor), so it is read-only here. A binding with no bindable selectable behaves as `Static`.

Use **Static** for visuals that should always use the same themed value: panel backgrounds, body text, icons, dividers, labels, decorative renderers, and any target that does not need hover/press/disabled feedback.

Use **Selectable** for controls that should mirror UI interaction state: buttons, dropdowns, sliders, tabs, toggles in their on state, and custom controls that implement [`IBindableSelectable`](ScriptingAPI.md#bindable-components). Each state row can point to a different definition, or several rows can intentionally share the same definition.

Use **Toggle** when an on/off control needs both selectable-state feedback and a separate visual set while off. This is useful for tabs, switches, segmented controls, and toggle buttons where "off + highlighted" should differ from "on + highlighted".

[![][2]][2]

## Selectable States

For `Selectable` and `Toggle` modes, a value can be set per state:

- `Normal`
- `Highlighted`
- `Pressed`
- `Selected`
- `Disabled`

Setting the **Normal** value first seeds the other states with the same value; adjust individual states afterwards as needed. In `Toggle` mode, the `Off Values` block exposes the same five states for the toggle's off state.

## Selecting a Value

Each state row is a themed value selector:

- **Definition dropdown** - lists the database definitions of the field's value type; pick one to store its GUID. The selection is shown by name, with a preview where available (a color dot, a sprite/texture thumbnail, or a value preview/tooltip).
- **Auto-selection** - a fresh field defaults to a value instead of `<None>` when the database can pick one for its type: the type's [favorite (default) value](ThemeDatabase.md#favorite-default-value-per-type) if one is set, otherwise the lone value when exactly **one** definition of that type exists.
- **Database shortcut** - the button next to the dropdown opens the active [`ThemeDatabase`](ThemeDatabase.md) for editing.

The value type is fixed by the binding (`TValue`), so the selector always knows what to offer.

Color values expose the same selector with an inline color preview, so state rows can be scanned without opening the database. The optional modifier row below each value lets the binding locally adjust the resolved value without creating another database definition.

[![][4]][4]

[![][3]][3]

## Add a Value from the Binding

If the active database has **no** value of the binding's type yet, the selector becomes an **`Add {Type} value…`** button. Clicking it:

1. adds a new definition of that exact type to the active database **as a constant value** (the simplest, immediately-resolvable form), seeded with the **target's current value** (for example the target graphic's current color) so adding a value preserves what the target already shows, and
2. selects the new definition on this field.

You can rename it, adjust its value, and - if it should differ per theme - convert it to a theme-specific value from the [database inspector](ThemeDatabase.md#constant-and-theme-specific-conversion).

## Value Modifiers

A [`ThemeValueModifier<T>`](ScriptingAPI.md#value-modifiers) can post-process the resolved value locally, without adding another database definition (the built-in `Color` modifiers cover override/multiply alpha and fade-to-opaque). The modifier UX is the same everywhere:

- **Hidden until a value is selected** - a modifier does nothing without a resolved value, so the row only appears once a definition is chosen.
- **No modifier for the type** - no row is shown; the raw value is used.
- **One or more modifiers** - a full-width dropdown offers `Unmodified` (the default, applies the resolved value as-is) plus every modifier available for the type. Pick one to adjust the value; its configuration is shown inline.
- **Renamed or removed** - the selected GUID is untouched and the raw value keeps resolving; a red recovery dropdown offers the remaining modifiers to re-select. See [Rename Safety](#rename-safety).

[![][5]][5]

## Apply To (Target)

- **Auto Target Discovery** (on by default) - resolves the `TTarget` automatically from this GameObject (or the GameObject itself for `GameObject` targets).
- **Target** - disable auto-discovery to assign the target manually.

Leave auto-discovery enabled for the common case where the binding sits beside the target component, for example a `BindColorToGraphic` on the same GameObject as an `Image`. Disable it when one binding should drive a specific child, a sibling component, or a renderer that cannot be found reliably from the binding's GameObject.

[![][6]][6]

## With Animation (Fade Duration)

Bindings whose `SetValueSmooth` provides a transition show a **Fade Duration**, a [`ThemedFloat`](ScriptingAPI.md#themedfloat): it is either a direct value or a themed `float` resolved from the active theme. The resolved duration is clamped to be non-negative; a duration of `0` (or a binding without smooth support) applies instantly.

[![][7]][7]

## Rename Safety

Renaming, moving, or removing a value type or modifier type never breaks a binding:

- The selected **definition GUID** is stored as plain data, so renaming or moving a value type keeps a binding's theme link intact.
- The optional **modifier** is stored by a rename-tolerant identifier plus a JSON config; a renamed or removed modifier keeps resolving the raw value and offers the remaining modifiers to re-select.

See [Scripting API - Rename Safety](ScriptingAPI.md#rename-safety).

## Related

- [Scripting API - Binding Architecture](ScriptingAPI.md#binding-architecture)
- [Scripting API - Built-in Bindings](ScriptingAPI.md#built-in-bindings)
- [Scripting API - Custom Binding Examples](ScriptingAPI.md#custom-binding-examples)
- [Getting Started - Add Theme Bindings](GettingStarted.md#add-theme-bindings)
- [ThemeDatabase](ThemeDatabase.md)

[1]: images/Bind_ValueDropdown_Full.png
[2]: images/BindColorToGraphic_Static.png
[3]: images/Bind_Value_Dropdown.png
[4]: images/BindColorToGraphic_Selectable.png
[5]: images/Bind_Modifiers.png
[6]: images/Bind_ApplyTo.png
[7]: images/Bind_WithAnimation.png
[8]: images/Bind_Type_Dropdown.png

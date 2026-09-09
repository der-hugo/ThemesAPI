Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - uGUI</h1>

`Themes` includes optional uGUI integration when Unity uGUI (`com.unity.ugui`) is installed: ready-made bindable wrappers for the standard `UnityEngine.UI.Selectable` controls, bindings that target uGUI/TMP components, a reference-safe conversion tool, and the `ThemeDropdown` helper. The core theming system works without uGUI - these are the pieces that light up when it is present.

## Contents

- [Requirements](#requirements)
- [What the Integration Adds](#what-the-integration-adds)
- [Bindable Components](#bindable-components)
- [Convert UI to Bindables](#convert-ui-to-bindables)
- [uGUI & TMP Bindings](#ugui--tmp-bindings)
- [Theme Switching (ThemeDropdown)](#theme-switching-themedropdown)
- [Usage Notes](#usage-notes)
- [Related](#related)

---

## Requirements

- `com.unity.ugui` installed in the project (provides `UnityEngine.UI`).
- TextMeshPro for the TMP-specific wrappers and bindings - bundled with uGUI `2.0.0`+, or the separate `com.unity.textmeshpro` package on older uGUI versions.

The uGUI bindings and bindable wrappers use version defines and compile automatically when the packages are present. Install them before opening content that uses their components; removing them from such a project leaves missing component scripts until they are reinstalled.

Without uGUI, the core still works: implement [`IBindableSelectable`](ScriptingAPI.md#bindable-components) on your own component to drive bindings, and target any component with a [custom binding](ScriptingAPI.md#custom-binding-examples).

## What the Integration Adds

Namespace: `derHugo.Themes`

- [Bindable component wrappers](#bindable-components) for standard `UI.Selectable` controls (`BindableButton`, `BindableToggle`, …).
- A [reference-safe conversion tool](#convert-ui-to-bindables) that replaces existing Unity selectables with their bindable variants.
- [uGUI & TMP bindings](#ugui--tmp-bindings) targeting `Graphic`, `Image`, `CanvasGroup`, `Selectable`, and `TMP_Text`.
- The [`ThemeDropdown`](#theme-switching-themedropdown) helper that binds a `TMP_Dropdown` to the available themes.

## Bindable Components

The built-in wrappers require uGUI and expose their state via `IBindableSelectable.State`. TMP wrappers additionally require TextMeshPro. They map directly to the standard Unity UI components and are normally added through the [conversion tool](#convert-ui-to-bindables) rather than by hand.

For frameworks other than uGUI, implement [`IBindableSelectable`](ScriptingAPI.md#bindable-components) on your own component instead - see the Scripting API for the interface contract.

Provided components:

- `BindableButton`
- `BindableToggle`
- `BindableSlider`
- `BindableScrollbar`
- `BindableDropdown` (*)
- `BindableInputField`
- `BindableTMPDropdown` (*)
- `BindableTMPInputField`

*&ast;: experimental - The Unity built-in states are behaving a bit weird for these*

Editor notes:

- Bindables force `transition = None` because themed bindings handle visual transitions.
- Bindable inspectors include an **Editor Preview State** debug control (edit mode only); `BindableScrollbar` uses the default inspector and does not show it.
- `BindableToggle` additionally exposes `onValueChangedInverted`.
- Each built-in selectable has a `Replace By Bindable{Type}` component context-menu entry (for example `Replace By BindableButton`) that runs the reference-safe conversion for that single component.

## Convert UI to Bindables

There are two ways to convert existing UI controls into their bindable variants. Both route through the same utility, which first scans the whole project (prefabs, ScriptableObjects and scenes) and rewires every existing reference to the replaced components, so **conversion never breaks existing references**.

### Replace a whole selection

Run the replacement utility on the selected roots. It converts every supported selectable found on the selected GameObjects and their children:

- `Tools > derHugo > Themes > Replace Selectables With Bindables (Selection)`
- or `GameObject > derHugo > Themes > Replace Selectables With Bindables (Selection)`

### Replace a single component (context menu)

Each supported built-in selectable also gets a dedicated entry in its component context menu (the `⋮` menu in the top-right of the component header, or right-click the component header). For example, a `Button` shows **Replace By BindableButton**:

- Right-click the component header (or open its `⋮` menu) and choose the `Replace By Bindable{Type}` entry.
- The entry only appears on the original Unity components; on components that are already bindable it is disabled.
- Selecting the entry on several objects at once batches them into a single conversion pass.

Supported replacements:

- `Button` -> `BindableButton`
- `Toggle` -> `BindableToggle`
- `Slider` -> `BindableSlider`
- `Scrollbar` -> `BindableScrollbar`
- `Dropdown` -> `BindableDropdown`
- `InputField` -> `BindableInputField`
- `TMP_Dropdown` -> `BindableTMPDropdown`
- `TMP_InputField` -> `BindableTMPInputField`

## uGUI & TMP Bindings

These [built-in bindings](ScriptingAPI.md#built-in-bindings) require uGUI (and, where noted, TextMeshPro). The full catalog - including the bindings that work **without** uGUI (`BindColorToRenderer`, `BindSpriteToSpriteRenderer`, `BindDepthToTransform`, `BindBoolToGameObjectActive`) - is in the [Scripting API](ScriptingAPI.md#built-in-bindings).

Require `com.unity.ugui`:

| Type                          | Target        |
|-------------------------------|---------------|
| `BindColorToGraphic`          | `Graphic`     |
| `BindColorBlockToSelectable`  | `Selectable`  |
| `BindMaterialToGraphic`       | `Graphic`     |
| `BindSpriteToImage`           | `Image`       |
| `BindFloatToCanvasGroupAlpha` | `CanvasGroup` |

Additionally require TextMeshPro:

| Type                         | Target     |
|------------------------------|------------|
| `BindColorToTMPText`         | `TMP_Text` |
| `BindStringToTextMeshPro`    | `TMP_Text` |
| `BindFontAssetToTMPText`     | `TMP_Text` |
| `BindFloatToTMPTextFontSize` | `TMP_Text` |

`BindColorToTMPText` and `BindFontAssetToTMPText` also drive 3D `TextMeshPro`. TextMeshPro comes bundled with uGUI `2.0.0`+; on older uGUI versions install `com.unity.textmeshpro` separately.

## Theme Switching (ThemeDropdown)

[`ThemeDropdown`](ScriptingAPI.md#theme-switching-helpers) binds a `TMP_Dropdown` to the available themes and the active theme selection, so it requires uGUI + TextMeshPro. The framework-independent switching helpers - [`ThemeSetter` and `ThemeSelector`](ScriptingAPI.md#theme-switching-helpers) - work without uGUI.

## Usage Notes

- Bindable wrappers set `transition = None`; drive their visuals through themed bindings instead.
- Convert existing selectables with the [conversion tool](#convert-ui-to-bindables) rather than swapping components by hand, so references are preserved.
- A binding auto-discovers its driving bindable selectable from the hierarchy (a bindable ancestor), so place bindings at or below the bindable selectable that should drive them.

## Related

- [Scripting API - Bindable Components](ScriptingAPI.md#bindable-components)
- [Scripting API - Built-in Bindings](ScriptingAPI.md#built-in-bindings)
- [Bind Components](Bind.md)
- [Getting Started](GettingStarted.md)
- [Localization](Localization.md)

Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - Getting Started</h1>

Use this guide for the first end-to-end setup pass.

## Contents

- [Create or Select the Active Database](#create-or-select-the-active-database)
- [Define Theme Values](#define-theme-values)
- [Configure Themes](#configure-themes)
- [Convert UI to Bindables](#convert-ui-to-bindables)
- [Add Theme Bindings](#add-theme-bindings)
- [Switch Themes at Runtime](#switch-themes-at-runtime)
- [Validate and Iterate](#validate-and-iterate)

---

## Create or Select the Active Database

Create a [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api):

- `Assets > Create > derHugo > Themes > ThemeDatabase`

Set it active in one of these places:

- [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) inspector (`Activate this Database`)
- `Window > derHugo > Themes`
- `Edit > Project Settings > derHugo > Themes`

If no database exists, the package will auto-create one at:

- `Assets/Resources/ThemeDatabase.asset`

You can freely move it somewhere else; the active-database link is tracked through preloaded assets, not its location.

More details: [ThemeDatabase - Active Database Lifecycle](ThemeDatabase.md#active-database-lifecycle)

## Define Theme Values

In the [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) inspector:

1. Add definitions in **Constant Values** or inside the **Themes** section.
2. Name each definition clearly (for example `Primary / Text / Accent`).
3. Pick the value type from the searchable type picker (`Color`, `Sprite`, `String`, `Float`, `Bool`, `Texture2D`, … - or any convertible type; a *Suggested* shortlist is offered first).
4. Decide whether it should be constant or theme-specific.

Definition selectors show previews where possible:

- `Color` entries show a color dot.
- `Sprite` and `Texture2D` entries show a thumbnail.
- Generic entries show the current value preview, with tooltips where additional detail is available.

Recommended first set:

- `Color`: surface, text, accent, warning
- `Sprite`: icon variants
- `string`: labels that are theme-variant
- `float`: depth/offset values

API details: [Scripting API - ThemeDatabase Runtime API](ScriptingAPI.md#themedatabase-runtime-api)  
More details: [ThemeDatabase - Value Definition Workflow](ThemeDatabase.md#value-definition-workflow)

## Configure Themes

1. Add a theme when you need theme-specific values, or keep a single default theme as a central style definition set.
2. Optionally add more themes (for example `Light`, `Dark`) from the **Themes** list `+` dropdown.
3. Fill theme-specific entries per theme.
4. Keep shared entries as constants.
5. Use the same `+` dropdown to import a single theme JSON file when needed.
6. Use a theme header's **Export JSON** button to export only that theme.
7. Set `Active Theme` and verify previews on bound components.

More details: [ThemeDatabase - Theme Workflow](ThemeDatabase.md#theme-workflow)

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

More details: [Scripting API - Bindable Components](ScriptingAPI.md#bindable-components)

## Add Theme Bindings

Add built-in binding components where needed:

- `BindColorToGraphic`
- `BindColorToRenderer`
- `BindSpriteToImage`
- `BindStringToTextMeshPro`
- `BindBoolToGameObjectActive`
- `BindDepthToTransform`

For most UI controls:

1. Assign the themed `Normal` value first (it seeds the other states).
2. Optionally override state-specific values (`Highlighted`, `Pressed`, `Selected`, `Disabled`).
3. For toggle-driven visuals, set binding `Type` to `Toggle` and provide `OffValues`.

If the database has no value of the needed type yet, the selector offers an **`Add {Type} value…`** button that creates one (as a constant, seeded with the target's current value) and selects it.

More details:

- [Bind Components and Inspector](Bind.md)
- [Scripting API - Built-in Bindings](ScriptingAPI.md#built-in-bindings)

## Switch Themes at Runtime

Common options:

- [`ThemeDropdown`](ScriptingAPI.md#theme-switching-helpers) (TMP dropdown based selector)
- [`ThemeSetter`](ScriptingAPI.md#theme-switching-helpers) with a serialized [`ThemeSelector`](ScriptingAPI.md#theme-switching-helpers)
- direct API call:

```csharp
ThemeDatabase.ActiveDatabase.SetActiveTheme(themeGuid);
```

More details:

- [Scripting API - Theme Switching Helpers](ScriptingAPI.md#theme-switching-helpers)
- [ThemeDatabase - Active Theme](ThemeDatabase.md#active-theme)

## Validate and Iterate

- Enter Play Mode and switch themes.
- Inspect bound controls for all selectable states.
- Export a full database JSON snapshot or individual theme JSON before large edits.
- Check [JSON Serialization Limitations](ThemeDatabase.md#json-serialization-limitations) before relying on JSON import/export for custom value types.

Related pages:

- [ThemeDatabase](ThemeDatabase.md)
- [Project Settings](ProjectSettings.md)
- [Sample](Sample.md)

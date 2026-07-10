Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - ThemeDatabase</h1>

This page documents the [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) workflow, data model, inspector behavior, and JSON snapshot format.

## Contents

- [Data Model](#data-model)
- [Active Database Lifecycle](#active-database-lifecycle)
- [Editor Entry Points](#editor-entry-points)
- [Inspector Workflow](#inspector-workflow)
- [Value Definition Workflow](#value-definition-workflow)
- [Constant and Theme-Specific Conversion](#constant-and-theme-specific-conversion)
- [Theme Workflow](#theme-workflow)
- [JSON Format and Import/Export](#json-format-and-importexport)
- [Rename Safety](#rename-safety)
- [JSON Serialization Limitations](#json-serialization-limitations)

---

## Data Model

[`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) stores:

- `ValueDefinitions`: schema entries (GUID, display name, value type, constant flag)
- `ConstantValues`: implementations for definitions marked constant
- `Themes`: named theme entries with per-theme values for non-constant definitions
- `ActiveThemeGuid`: active theme identifier

Runtime references ([`ThemeValue<T>`](ScriptingAPI.md#themevalue-and-value-types)) point to definitions by GUID.

## Active Database Lifecycle

### Single active database

Only one database should be active project-wide.

The active database is selected through the editor tooling and synced to `PlayerSettings` preloaded assets so runtime can resolve the same asset.

### Automatic creation

If no database exists, the editor can create:

- `Assets/Resources/ThemeDatabase.asset`

`Resources` is only the default creation location. The active database is resolved through the `PlayerSettings` preloaded assets, so you can move the asset out of `Resources` to any folder or package without breaking the active-database link. See [Getting Started - Create or Select the Active Database](GettingStarted.md#create-or-select-the-active-database).

### Multiple database handling

If multiple databases exist and no clear active selection is available, the editor prompts you to choose one.

### Switch confirmation

When switching from one active database to another, a confirmation dialog warns that bindings may break due to different definition GUID sets.

### Runtime-facing state

[`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) publishes:

- [`ThemeDatabase.ActiveDatabase`](ScriptingAPI.md#themedatabase-runtime-api)
- [`ThemeDatabase.ActiveTheme`](ScriptingAPI.md#themedatabase-runtime-api)
- [`ThemeDatabase.AvailableThemes`](ScriptingAPI.md#themedatabase-runtime-api)

`ActiveThemeGuid` is validated; if invalid or empty, the first theme becomes active.

## Editor Entry Points

- [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) asset inspector
- `Window > derHugo > Themes`
- `Edit > Project Settings > derHugo > Themes`

All entry points render the same core editor workflow.

## Inspector Workflow

Shared window/settings header:

- **Project-wide Database** selector
- **Export JSON** and **Import JSON** actions

Asset inspector header:

- **Export JSON** for the inspected database

Main sections:

1. **Active Theme** popup
2. **Constant Values** list
3. **Themes** list with nested per-theme value rows

If the inspected database is not active, the inspector shows:

- `Activate this Database`

Dropdowns and selector rows include previews where the value can be resolved:

- `Color` entries show a single-line color dot.
- `Sprite` and `Texture2D` entries show a single-line thumbnail.
- Generic entries show the current value preview, with tooltips where additional detail is available.

[![][1]][1]

The editor window is optimized for two common tasks:

- Use **Constant Values** to maintain shared design tokens that do not vary between themes, such as numeric spacing, fixed durations, asset references, or a default sprite.
- Use **Themes** to compare and edit every theme-specific implementation of the same definition, such as light/dark colors, theme names, localized labels, state colors, and layout values that should change with the active theme.
- Use **Active Theme** while authoring a scene to preview the result immediately in bound components.
- Use **Export JSON** when handing a database to another project, reviewing a theme in version control, or backing up a known-good set of definitions before a larger edit.

## Value Definition Workflow

Definitions can be added from:

- **Constant Values** list `+` (add as constant)
- nested **Themes** value list `+` (add as theme-specific)

Adding or changing a type opens a **searchable type picker**: a *Suggested* shortlist (built-in types, types with a registered `ThemeValueJsonConverter<T>` or `ThemeValueModifier<T>`, and types already used in this database) plus a search across every convertible type. Built-in base types are shown by their common names (`Int`, `Float`, `Bool`, …). Collections and generic type instances such as `List<Color>`, `Color[]`, or `Dictionary<string, Color>` are not added directly through the picker; wrap them in a named `[Serializable]` type first. See [Scripting API - ThemeValue and Value Types](ScriptingAPI.md#themevalue-and-value-types).

For each definition:

- Edit display name
- Change value type (via the type picker)
- Reorder within its constant or theme-specific group
- Remove the definition

Important behavior:

- Changing a definition type resets serialized values for that definition across all themes.
- GUID-based references remain stable when renaming or reordering.
- If a definition's value type can no longer be resolved (renamed/removed script), its type dropdown turns red and the stored value is preserved until you pick a replacement type - see [Rename Safety](#rename-safety).

[![][2]][2]

The value type controls which bindings can consume the definition:

- `Color` definitions feed color bindings such as `BindColorToGraphic` and `BindColorToRenderer`.
- `Sprite` and `Texture2D` definitions feed image/texture bindings and show asset previews in the inspector.
- `Float`, `Int`, `Bool`, `String`, vectors, enums, and serializable custom structs can be used by built-in or custom bindings.
- Object references are useful for shared assets, but should point to project or package assets when JSON portability matters. This is an editor feature and will not work for side loading on runtime!

Name definitions by intent rather than by one theme's implementation. For example, prefer `Primary Color`, `Dialog Background`, or `Button Fade Duration` over `Blue`, `Dark Gray`, or `0.2`. Bindings store the definition GUID, so the display name can be refined later without losing references.

## Constant and Theme-Specific Conversion

Definitions can be converted both ways:

- Theme-specific -> constant
- Constant -> theme-specific

Conversion behavior:

- Theme-specific -> constant keeps one value as baseline.
- If themes currently differ, conversion asks for confirmation.
- Constant -> theme-specific clones the constant value into each theme.

## Theme Workflow

Theme operations:

- Add theme from the **Themes** list `+` dropdown
- Import one theme from the **Themes** list `+` dropdown
- Rename theme
- Reorder themes
- Export one theme from its header `Export JSON` button
- Delete theme (disabled when only one theme remains)
- Set active theme through the `Active Theme` popup

Each theme stores implementations only for non-constant definitions.

### Active Theme

`ActiveThemeGuid` selects the currently active theme entry.

If `ActiveThemeGuid` is empty or no longer points to a valid theme, the first theme in the list is used.

## JSON Format and Import/Export

[`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) supports export/import as JSON snapshots for complete databases and individual themes.

[![][3]][3]

### Complete database JSON

Top-level **Export JSON** writes a complete database snapshot. Top-level **Import JSON** imports a complete database snapshot into the current database or into a newly created database asset.

Shape:

```json
{
  "formatVersion": 2,
  "activeThemeGuid": "6ef48aed5f6e4964bd91861511cb9f1a",
  "valueDefinitions": {
    "0904b5ff1d574b99ad767ef3e559f903": {
      "displayName": "Primary Color",
      "type": "UnityEngine.Color, UnityEngine.CoreModule",
      "isConstant": false
    }
  },
  "constantValues": {},
  "themes": {
    "674a69f43669460097cda2f9bab85253": {
      "displayName": "Light",
      "values": {
        "0904b5ff1d574b99ad767ef3e559f903 Primary Color": "#FFFFFFFF"
      }
    }
  }
}
```

Key and serialization notes:

- Value entries in `constantValues` and `themes.*.values` are exported as `<definitionGuid> <displayName>` for hand-editing readability. Import only uses the GUID before the first space. The displayName is rather taken from the ValueDefinitions
- Value conversion is keyed by value type. A registered [`ThemeValueJsonConverter<T>`](ScriptingAPI.md#custom-json-serialization) (custom or built-in) converts the value; otherwise Unity-like field serialization (matching `JsonUtility`'s rules) is used.
- Built-in converters serialize plain scalars as JSON primitives, `Color` and `ColorBlock` as HEX (`#RRGGBBAA`), enums by name, and `Quaternion` as Euler-angle objects.
- `formatVersion` identifies the payload format; the current format is `2`.
- `UnityEngine.Object` references are serialized as asset path strings. This works within the editor but not for side loading Themes on runtime.
- Sub-assets are serialized as `path#ObjectName`.

Import modes:

- **Import Into This Database**: destructive overwrite of current content.
- **Import As New Database Asset**: creates a new asset for review.

### Individual theme JSON

Use a theme header's **Export JSON** button to write only that theme. Use the **Themes** list `+` dropdown > **Import Theme** to add one theme JSON file to the current database.

Shape:

```json
{
  "formatVersion": 2,
  "displayName": "Dark",
  "guid": "6ef48aed5f6e4964bd91861511cb9f1a",
  "values": {
    "0904b5ff1d574b99ad767ef3e559f903 Primary Color": "#111111FF"
  }
}
```

Theme import modes:

- **Import And Activate**: adds the imported theme and makes it active.
- **Import Only**: adds the imported theme without switching the active theme.

Theme import review behavior:

- If the imported theme GUID already exists, the editor asks whether to override it or add a new theme with a generated GUID.
- Unknown value GUIDs are ignored with warnings because the target value type is unknown.
- Values for constant definitions can optionally convert those definitions to theme-specific values.
- Missing themed values keep their default implementations.

Import diagnostics:

- Invalid structure/types produce errors and abort import.
- Unknown definition GUID references in values are skipped with warnings.

## Rename Safety

`ThemeDatabase` stores its definitions and values as **plain data**: each definition keeps a GUID, a display name, a constant flag, and a rename-tolerant value-type identifier (`Namespace.Type, Assembly`), and each stored value is a JSON payload plus an object side-list.

Because of this, renaming, moving, or removing a value type's script keeps the stored data intact:

- The database keeps the stored value; the inspector shows a red type dropdown (the searchable picker) to re-point the definition to a resolvable type, preserving the value.
- Assembly moves and version bumps resolve automatically (the identifier is matched by full name across loaded assemblies).

See [Scripting API - Rename Safety](ScriptingAPI.md#rename-safety) for the consumer/`ThemeValue<T>` side.

## JSON Serialization Limitations

JSON import/export is intended for theme database transfer and backup, not as a general-purpose serializer for every possible value type.

Conversion order (keyed by value type):

1. A registered [`ThemeValueJsonConverter<T>`](ScriptingAPI.md#custom-json-serialization) (custom or built-in) converts the value.
2. `UnityEngine.Object` values use editor asset paths (sub-assets as `path#ObjectName`) via the object side-list.
3. Remaining values use Unity-like field serialization (matching `JsonUtility`'s rules).

Built-in converter shapes (examples - see [Sample - JSON Converter References](Sample.md#json-converter-references) for the full list):

- `Color` as HEX strings (`#RRGGBBAA`).
- `ColorBlock` as `normalColor`, `highlightedColor`, `pressedColor`, `selectedColor`, and `disabledColor` HEX strings plus `colorMultiplier` and `fadeDuration`.
- plain scalars such as `float`, `int`, `bool`, and `string` as JSON primitives, and enums as string names.
- `Quaternion` as Euler angles, for example `{ "x": 12.3, "y": 4.56, "z": 2.5 }`.

Fallback value shapes:

- `[Serializable]` structs/classes and Unity value types that Unity can serialize are converted with Unity-like field serialization (matching `JsonUtility`'s rules).
- The exact JSON field names and structure for fallback values follow Unity's serializer.

Known limitations:

- Types with no serializable fields (that Unity-like field serialization would emit as empty output, `""` or `{}`) require a custom `ThemeValueJsonConverter<T>`.
- Scene object references are not portable JSON references. `UnityEngine.Object` support is asset-path based and expects the referenced asset to exist in the target project.
- Runtime single-theme import cannot resolve `UnityEngine.Object` references; object reference import is editor-only.
- Individual theme import requires matching value definitions so the expected target type is known.

Recommended approach for custom values:

- Prefer plain scalar values, enums, `Color`, supported Unity value structs, asset references, or `[Serializable]` DTOs that Unity can serialize directly.
- For custom payloads, prefer `[Serializable]` classes/structs made of public fields, `[SerializeField]` fields, and supported nested values.
- Add explicit JSON support when a type needs a stable, hand-authored JSON schema or is not supported by Unity's `JsonUtility`.

Related pages:

- [Getting Started](GettingStarted.md)
- [Project Settings](ProjectSettings.md)
- [Scripting API](ScriptingAPI.md)
- [Sample](Sample.md)

[1]: images/ThemeDatabase_Full.png
[2]: images/ThemeDatabase_ValueTypes_Full.png
[3]: images/JSON_Serialization_Full.png

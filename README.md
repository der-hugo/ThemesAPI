<h1><img src="Documentation~/images/colors.png" alt="Logo" width="120" align="middle" />&nbsp; Themes</h1>

Themes is a [`ThemeDatabase`](Documentation~/ScriptingAPI.md#themedatabase-runtime-api)-driven UI value and binding system for theming, white-label branding, and style profile switching.

It centralizes visual and behavioral UI values (colors, sprites, strings, floats, bools, textures), then applies them consistently across complex hierarchies in edit mode and runtime.

## Contents

- [Where do I get it?](#where-do-i-get-it)
- [Requirements & Dependencies](#requirements--dependencies)
- [Highlights](#highlights)
- [Core Concepts](#core-concepts)
- [Typical Workflow](#typical-workflow)
- [Documentation](#documentation)

## Where do I get it?

Please find the latest version of `Themes` on the [Unity Asset Store](https://assetstore.unity.com/packages/slug/390596).

## Requirements & Dependencies

- Unity `6000.3` or newer
- `com.unity.ugui`
- `com.unity.nuget.newtonsoft-json`
- `com.unity.textmeshpro`
- [optional] `com.unity.localization` (adds a LocalizedString -> TMP_Text binding)

The included [Sample](Documentation~/Sample.md) has additional dependencies which are intentionally not installed via asset manifest.

## Highlights

- Centralized [`ThemeDatabase`](Documentation~/ScriptingAPI.md#themedatabase-runtime-api) system
- Theme **any serializale value** via [`ThemeValue<T>`](Documentation~/ScriptingAPI.md#themevalue-and-value-types) - extensible via converters, modifiers, and previews.
- [`Bindable` wrappers](Documentation~/ScriptingAPI.md#bindable-components) for standard selectables (`Button`, `Toggle`, `Slider`, `Scrollbar`, `Dropdown`, `InputField`, `TMP_Dropdown`, `TMP_InputField`).
- [Built-in bindings](Documentation~/ScriptingAPI.md#built-in-bindings) for common UI targets (`Graphic`, `Image`, `TMP_Text`, `Transform`, `GameObject`), with extension points for arbitrary custom targets.
- Full selectable-state hook for fan-out binding: one selectable can drive any number of bindings and update multiple graphics/components per state.
- Inspector dropdown previews for theme values, including color dots, sprite/texture thumbnails, and value previews/tooltips for generic entries.
- Editor JSON import/export for full database snapshots and Runtime side loading of individual themes.

## Core Concepts

### Active ThemeDatabase

The package works with exactly one active [`ThemeDatabase`](Documentation~/ScriptingAPI.md#themedatabase-runtime-api) per project session.

When no database exists, the editor creates a default one at:

    Assets/Resources/ThemeDatabase.asset

`Resources` is only the default creation location, not a requirement. The active database is resolved through `PlayerSettings` preloaded assets, not `Resources.Load`, so it does not need to stay in a `Resources` folder (where it would otherwise be force-included in every build).

### Value Definitions and GUID Binding

Each definition has a stable GUID and a value type. Runtime [`ThemeValue<T>`](Documentation~/ScriptingAPI.md#themevalue-and-value-types) references store this GUID and resolve the value through the active database.

### Constant vs Theme-Specific Values

- **Constant values** are shared across all themes.
- **Theme-specific values** are stored separately in each theme.

Definitions can be converted both ways directly in the inspector.

### Bindables and Bindings

- [**Bindable components**](Documentation~/ScriptingAPI.md#bindable-components) expose selectable state changes as observable state.
- [**Binding components**](Documentation~/ScriptingAPI.md#binding-architecture) read [`ThemeValue<T>`](Documentation~/ScriptingAPI.md#themevalue-and-value-types) and apply values to target components for static, selectable, or toggle-aware workflows.
- Built-in bindings focus on uGUI/TMP targets, but custom bindings can target arbitrary component types (for example `Light` in the sample).

## Typical Workflow

1. Create or select the active [`ThemeDatabase`](Documentation~/ScriptingAPI.md#themedatabase-runtime-api).
2. Define themed entries and decide constant vs per-theme storage.
3. Configure your theme set (for example Light and Dark).
4. [Replace selectables with bindable variants](Documentation~/GettingStarted.md#convert-ui-to-bindables) via the selection menu or a component's `Replace By Bindable{Type}` context menu (references are preserved).
5. Add bindings and assign [`ThemeValue<T>`](Documentation~/ScriptingAPI.md#themevalue-and-value-types) references.
6. Switch themes through [`ThemeDropdown` / `ThemeSetter` / `ThemeSelector`](Documentation~/ScriptingAPI.md#theme-switching-helpers), or direct API calls.

## Documentation

- [Getting Started](Documentation~/GettingStarted.md): first setup pass and quick tutorial.
- [ThemeDatabase](Documentation~/ThemeDatabase.md): in-depth data model, inspector workflow, JSON format, rename safety, and serialization limitations.
- [Bind Components](Documentation~/Bind.md): the binding components and their inspector - value selection, modifiers, quick-add, targets, and transitions.
- [Project Settings](Documentation~/ProjectSettings.md): project-wide activation and settings behavior.
- [Sample](Documentation~/Sample.md): walkthrough of the included example content.
- [Scripting API](Documentation~/ScriptingAPI.md): public API usage with focus on custom value types, converters, modifiers, previews, custom bindings, and rename safety.
- [Localization](Documentation~/Localization.md): optional localization integration.

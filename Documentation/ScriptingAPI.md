Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - Scripting API</h1>

This page documents the public authoring surface with a focus on themed value types, the value extension points, and custom bindings.

## Contents

- [Overview](#overview)
- [ThemeDatabase Runtime API](#themedatabase-runtime-api)
- [JSON Import/Export API](#json-importexport-api)
- [ThemeValue and Value Types](#themevalue-and-value-types)
- [ThemedFloat](#themedfloat)
- [Value Modifiers](#value-modifiers)
- [Binding Architecture](#binding-architecture)
- [Built-in Bindings](#built-in-bindings)
- [Bindable Components](#bindable-components)
- [Theme Switching Helpers](#theme-switching-helpers)
- [Value Previews](#value-previews)
- [Custom JSON Serialization](#custom-json-serialization)
- [Rename Safety](#rename-safety)
- [Custom Binding Examples](#custom-binding-examples)
- [Optional Localization API](#optional-localization-api)
- [JSON Serialization Limitations](#json-serialization-limitations)
- [Choose the Right Extension Point](#choose-the-right-extension-point)

---

## Overview

`ThemeValue<T>` is a concrete, ready-to-use type: add `[SerializeField] ThemeValue<MyType> field;` to any component and it resolves a themed value of `MyType`, just like using `List<T>`. Any type the database can store works with **no subclassing**.

You extend the package through small, focused hooks:

- [`ThemeValueJsonConverter<T>`](#custom-json-serialization) - a registered, hand-authored JSON shape for a value type, applied recursively wherever the type appears.
- [`ThemeValueModifier<T>`](#value-modifiers) - optional local post-processing of a resolved value.
- [`ThemeValuePreview<T>`](#value-previews) *(editor)* - a compact inspector swatch for a value type.
- [`Bind<TValue, TTarget>`](#binding-architecture) - apply themed values to a custom runtime target.

## ThemeDatabase Runtime API

Namespace: `derHugo.Themes`

Key members:

- `ThemeDatabase.ActiveDatabase` (static `ThemeDatabase`) - the resolved active database.
- `ThemeDatabase.ActiveTheme` (static `ReadOnlyObservableWithState<Theme>`) - read `.Value` for the current theme, or subscribe to react to theme changes.
- `ThemeDatabase.AvailableThemes` (static `IReadOnlyList<Theme>`) - all themes in the active database.

Theme switching (instance methods on the active database; each returns `bool`):

- `SetActiveTheme(int index)`
- `SetActiveTheme(Theme theme)`
- `SetActiveTheme(string themeGuid)`

Value access:

- `TryGetValue<T>(string definitionGuid, out T value)` - resolves a definition through the active theme; returns `false` when it can not be resolved.
- `TryGetValueByKey<T>(string key, out T value)` - resolves an [alias](Aliases.md) key to its target definition, then through the active theme; returns `false` when the alias is missing, unmapped, or resolves to an incompatible type.

Example:

```csharp
using derHugo.Themes;
using UnityEngine;

public sealed class ThemeApiExample : MonoBehaviour
{
    [SerializeField] private string _themeGuid;

    public void ApplyTheme()
    {
        ThemeDatabase.ActiveDatabase?.SetActiveTheme(_themeGuid);
    }
}
```

## JSON Import/Export API

Namespace: `derHugo.Themes`

Editor-only complete database helpers:

- `ExportJson()`: exports the complete database JSON snapshot.
- `TryImportJson(string json, out string message)`: imports a complete database snapshot into this database.
- `TryCreateDatabaseAssetFromJson(string json, string assetPath, out ThemeDatabase database, out string message)`: creates a new database asset from a complete database snapshot.

Editor-only individual theme helper:

- `ExportThemeJson(Theme theme)`: exports a single theme JSON file that can be re-imported through the inspector.

Runtime/editor individual theme import:

- `TryAddThemeFromJson(string json, out string themeGuid, out string message, bool setActive = false)`

Notes:

- Complete database JSON import/export is editor-only.
- Individual theme import is available at runtime, but runtime import cannot resolve `UnityEngine.Object` references.
- The editor import path includes review dialogs for GUID conflicts, missing definitions, and constant-vs-themed mismatches.
- See [ThemeDatabase - JSON Format and Import/Export](ThemeDatabase.md#json-format-and-importexport) for the JSON shapes.

## ThemeValue and Value Types

Namespace: `derHugo.Themes`

`ThemeValue<T>` is a concrete, plain-serialized reference to a themed value. It **needs no subclass** - use `ThemeValue<Color>`, `ThemeValue<float>`, `ThemeValue<MyType>` directly on any component.

Serialized data:

- `DefinitionGuid` - the selected value definition, stored as plain data (survives any type rename/move).
- `Key` - an optional [alias](Aliases.md) key; when set it takes precedence over `DefinitionGuid` and resolves the value by name instead of GUID (so a pre-configured binding can resolve against a consuming project's database).
- an optional [`ThemeValueModifier<T>`](#value-modifiers), stored by a rename-tolerant type identifier plus a JSON config blob.

Key behavior:

- `TryGetValue(out T value)` resolves the reference through the active database, then applies the local modifier when one is set. A missing/unresolved modifier degrades to the raw value - the theme link is never lost.
- When a `Key` is set, resolution goes through the database's [alias](Aliases.md) map to the target definition first; an unresolved key (no alias, unmapped, or a type mismatch) keeps the field's current value rather than clearing it.

Available value types:

- Any type the database can serialize is available - primitives, enums, Unity value types (`Color`, `Vector*`, `Quaternion`, `Rect`, `Bounds`, …), `UnityEngine.Object` subtypes (`Sprite`, `Texture2D`, `GameObject`, …), and your own `[Serializable]` types.
- Pick the type from the database's **searchable value-type picker** when adding a value; built-in and already-used types appear as a "Suggested" shortlist, and search covers any convertible type. See [ThemeDatabase](ThemeDatabase.md).
- Collections and generic type instances such as `List<Color>`, `Color[]`, or `Dictionary<string, Color>` are not added directly through the type picker. Wrap them in a named `[Serializable]` type first, then use `ThemeValue<YourWrapper>` and add a converter when the wrapper needs a hand-authored JSON shape.
- A type needs a [`ThemeValueJsonConverter<T>`](#custom-json-serialization) only if you want a specific JSON shape for import/export; otherwise Unity-like field serialization (matching `JsonUtility`'s rules) handles it.

## ThemedFloat

Namespace: `derHugo.Themes`

`ThemedFloat` is a serializable `float` that can either hold a direct custom value or resolve a
themed `float` definition through the active database. It lets a single field
flip between a hard-coded value and a value driven by the current theme.

```csharp
public enum Source
{
    Custom = 0,
    Themed = 1
}
```

| Member                                  | Description                                                                                  |
|-----------------------------------------|----------------------------------------------------------------------------------------------|
| `Mode`                                  | `Custom` uses `CustomValue`; `Themed` resolves `DefinitionGuid` through the active database. |
| `CustomValue`                           | The direct value used in `Custom` mode.                                                      |
| `DefinitionGuid`                        | The `float` definition GUID resolved in `Themed` mode.                                       |
| `TryGetValue(out float value)`          | Resolves the effective value; returns `false` when a themed value can not be resolved.       |
| `GetValueOrDefault(float fallback = 0)` | Resolves the effective value, or `fallback` when a themed value can not be resolved.         |

Used by:

- the built-in `Color` modifiers' `Factor` (Override Alpha / Multiply Alpha / Fade Opaque)
- `Bind<TValue, TTarget>.FadeDuration`

Editor behavior:

- In `Custom` mode, the inspector edits the direct value.
- In `Themed` mode, the inspector shows the selected `float` definition and a read-only preview of the currently resolved theme value.

## Value Modifiers

Namespace: `derHugo.Themes`

A modifier post-processes a resolved value locally, per `ThemeValue<T>` reference - without adding another database definition. Each modifier is one concrete type: implement `ThemeValueModifier<T>` for the value type you want to transform, override `Modify(ref T value)`, and add `[SerializeField]` fields for its configuration.

Every concrete `ThemeValueModifier<T>` is discovered automatically by its value type `T` and offered in the inspector. Selection is intentionally opt-in:

- **No modifier for the type** - no modifier row is shown at all; the reference resolves the raw value directly.
- **One or more modifiers** - a full-width dropdown lists `Unmodified` (the default) plus every modifier available for the type. The row is hidden until a value is selected (a modifier does nothing without a resolved value).
- **`Unmodified`** - the resolved value is applied as-is, without any modifier. This is the common case.
- **A modifier selected** - its `[SerializeField]` configuration is drawn inline (through its own `[CustomPropertyDrawer]` when it has one, otherwise the default inspector) and stored as a small JSON blob.

A modifier is stored by a rename-tolerant type identifier plus that JSON config, so renaming, moving, or removing a modifier type keeps the reference working: the owning `ThemeValue<T>` keeps its definition GUID, resolves the raw value, and offers the remaining modifiers to re-select. See [Rename Safety](#rename-safety).

```csharp
[Serializable]
public sealed class GrayscaleColorModifier : ThemeValueModifier<Color>
{
    [field: SerializeField] [field: Range(0f, 1f)]
    public float Amount { get; private set; } = 1f;

    public override void Modify(ref Color value)
    {
        var gray = value.grayscale;
        value = Color.Lerp(value, new Color(gray, gray, gray, value.a), Mathf.Clamp01(Amount));
    }
}
```

Model each distinct behavior as its own modifier type rather than one type with an internal "mode" enum: each behavior then appears as its own dropdown entry, shows only the fields it actually uses, and leaving a reference `Unmodified` cleanly means "no modifier" instead of selecting a no-op mode.

The built-in `Color` modifiers follow this pattern - each is a separate, independently selectable `ThemeValueModifier<Color>`:

- **Override Alpha** - replaces the resolved alpha with `Factor`.
- **Multiply Alpha** - multiplies the resolved alpha by `Factor`.
- **Fade Opaque** - fades the color toward an opaque reference by `Factor`, against `AgainstWhite`, `AgainstBlack`, `AgainstSelected` (another color definition by GUID), or `AgainstCustom`.

In each, `Factor` is a [`ThemedFloat`](#themedfloat) (custom or themed `float`), clamped to `[0,1]` after resolution (an unresolved themed factor falls back to `1`); an unresolved `AgainstSelected` falls back to white.

## Binding Architecture

Base classes:

- `Bind<TValue, TTarget>`

`Bind<TValue, TTarget>` applies values for selectable state variants:

- `Normal`
- `Highlighted`
- `Pressed`
- `Selected`
- `Disabled`

Binding type modes:

- `Static`: uses only static value path
- `Selectable`: responds to selectable state transitions
- `Toggle`: also supports `OffValues` when the bound toggle is off

`Bind<TValue, TTarget>` also exposes `FadeDuration` (a [`ThemedFloat`](#themedfloat), so it can be a custom value or a themed `float`) for bindings whose `SetValueSmooth` returns a routine. The resolved duration is never negative. Returning `null` falls back to `SetValue`.

A concrete binding implements two abstract members:

- `SetValue(TValue value, TTarget target)` - applies a resolved value to the target.
- `GetValue(TTarget target)` - reads the target's current value (the inverse of `SetValue`). The inspector's quick-add uses it to seed a newly created theme value with the target's current state.

Optionally override `SetValueSmooth` for animated transitions.

For the inspector workflow - value selection, single-value auto-default, modifiers, the `Add {Type} value…` quick-add, targets, and transitions - see [Bind Components and Inspector](Bind.md).

## Built-in Bindings

Bindings targeting uGUI (`Graphic`, `Image`, `Selectable`, `CanvasGroup`) require `com.unity.ugui`. TMP bindings additionally require TextMeshPro, bundled with uGUI 2.0.0+ or supplied by `com.unity.textmeshpro`. Other bindings and `Bind<TValue, TTarget>` remain available without these packages. The table below is the complete catalog; for the uGUI/TMP subset and its requirements see [uGUI - uGUI & TMP Bindings](uGUI.md#ugui--tmp-bindings).

Namespace: `derHugo.Themes`

| Type                          | Target           | Notes                                                                                                                                                                                                                                                                                                                                                     |
|-------------------------------|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `BindColorToGraphic`          | `Graphic`        | Supports smooth fade using `FadeDuration`.                                                                                                                                                                                                                                                                                                                |
| `BindColorToTMPText`          | `TMP_Text`       | Drives `TMP_Text.color` directly, so it also colors the 3D `TextMeshPro` (which `BindColorToGraphic` cannot reach). Supports smooth fade using `FadeDuration`.                                                                                                                                                                                            |
| `BindColorToRenderer`         | `Renderer`       | Tints any renderer via a `MaterialPropertyBlock` (`_BaseColor`/`_Color`/`_TintColor`), so no material instance is created. Also drives `LineRenderer`/`TrailRenderer` start/end colors. Works under URP where vertex colors are ignored. Supports smooth fade using `FadeDuration`.                                                                       |
| `BindColorBlockToSelectable`  | `Selectable`     | Applies a themed `ColorBlock` to `Selectable.colors` (Unity's built-in color-tint transition). Best on selectables that keep Unity's `ColorTint` transition rather than per-state color bindings.                                                                                                                                                         |
| `BindMaterialToRenderer`      | `Renderer`       | Switches the renderer's primary (slot 0) material to the themed **shared** asset - never instances, so it is safe in edit mode and leaks nothing. With `FadeDuration` at runtime it blends shader properties via `Material.Lerp` (only when both materials share a shader; otherwise it switches instantly). Other sub-material slots are left untouched. |
| `BindMaterialToGraphic`       | `Graphic`        | UI counterpart to `BindMaterialToRenderer`: switches (or blends via `Material.Lerp`) `Graphic.material` to the themed **shared** asset. A `Graphic` never instances its material, so nothing leaks.                                                                                                                                                       |
| `BindSpriteToImage`           | `Image`          | Applies sprites by selectable state.                                                                                                                                                                                                                                                                                                                      |
| `BindSpriteToSpriteRenderer`  | `SpriteRenderer` | 2D world-space counterpart to `BindSpriteToImage`.                                                                                                                                                                                                                                                                                                        |
| `BindStringToTextMeshPro`     | `TMP_Text`       | Applies text strings.                                                                                                                                                                                                                                                                                                                                     |
| `BindFontAssetToTMPText`      | `TMP_Text`       | Swaps the `TMP_FontAsset` (shared asset reference, no instancing). Works for `TextMeshProUGUI` and 3D `TextMeshPro`.                                                                                                                                                                                                                                      |
| `BindFloatToTMPTextFontSize`  | `TMP_Text`       | Applies font size.                                                                                                                                                                                                                                                                                                                                        | 
| `BindBoolToGameObjectActive`  | `GameObject`     | Toggles active state.<br/>**NOTE:** Since the subscription is cancelled on disabling the component it is recommended to drive this from the button not to the target `GameObject` itself.                                                                                                                                                                 |
| `BindDepthToTransform`        | `Transform`      | Applies local `z` value with optional smoothing.                                                                                                                                                                                                                                                                                                          |
| `BindFloatToCanvasGroupAlpha` | `CanvasGroup`    | Applies a `[0,1]`-clamped `float` to `CanvasGroup.alpha` (fade whole UI groups by state). Supports smooth fade using `FadeDuration`.                                                                                                                                                                                                                      |


## Bindable Components

Namespace: `derHugo.Themes`

A bindable selectable exposes its interaction state as an observable that bindings subscribe to. This is a **core** concept: the interfaces live in the core assembly and work without uGUI. The ready-made wrappers for standard Unity UI controls (`BindableButton`, `BindableToggle`, …), their editor preview, and the conversion tool are part of the optional [uGUI integration](uGUI.md#bindable-components).

To drive bindings from your own (non-uGUI) component, implement `IBindableSelectable`: expose a `ReadOnlyObservableWithState<IBindableSelectable.StateInfo>` backed by an `ObservableWithState<IBindableSelectable.StateInfo>` and publish updates using `SetValue(new IBindableSelectable.StateInfo(state, instant))`. Implement `IBindableToggle` with an `isOn` property for on/off value blocks; the binding inspector recognizes this interface, not just the built-in `BindableToggle`.

`BindableSelectable.BindableSelectionState` is framework-independent and retains its serialized values: Normal = 0, Highlighted = 1, Pressed = 2, Selected = 3, Disabled = 4. Do not inherit from the compatibility bridge `BindableSelectable` itself.

Editor preview is optional for custom implementations: `UseEditorPreviewState` and `EditorPreviewState` have no-op defaults. Override both properties under `#if UNITY_EDITOR` if your custom inspector supports preview simulation.

For the ready-made uGUI wrappers and how to convert existing selectables to them, see [uGUI - Bindable Components](uGUI.md#bindable-components).

## Theme Switching Helpers

Namespace: `derHugo.Themes`

- `ThemeSelector`: serializable GUID wrapper with `TryApply()`.
- `ThemeSetter`: Uses an exposed `ThemeSelector` and applies the selected theme on enable (gated by `Execute In Edit Mode` / `Apply On Enabled`) or by explicit `Apply()` call.
- `ThemeDropdown`: binds a `TMP_Dropdown` to available themes and active theme selection. Requires the optional [uGUI integration](uGUI.md#theme-switching-themedropdown) (uGUI + TextMeshPro)

![ThemeSetter inspector](images/sample/ThemeSetter.png)

## Value Previews

Namespace: `derHugo.Themes.Editor`

A preview draws the compact inspector swatch for a value type - the small color dot, or the sprite/texture thumbnail shown left of the field and inside the value dropdown. Built-ins cover `Color`, `Sprite`, and `Texture2D`. Add your own for a custom type by subclassing `ThemeValuePreview<T>` and marking it with `[ThemeValuePreview]`; it is discovered automatically. Types without a preview fall back to Unity's native read-only field (which honors any `[CustomPropertyDrawer]` you wrote for the type), so previews are purely additive.

```csharp
using derHugo.Themes.Editor;
using UnityEditor;
using UnityEngine;

[ThemeValuePreview]
public sealed class Vector2Preview : ThemeValuePreview<Vector2>
{
    public override bool DrawSwatch(Rect rect, Vector2 value)
    {
        EditorGUI.LabelField(rect, $"({value.x:0.#}, {value.y:0.#})");
        return true;
    }
}
```

## Custom JSON Serialization

Namespace: `derHugo.Themes`

JSON conversion is keyed by **value type** and runs through a single recursive [Newtonsoft](https://www.newtonsoft.com/json) serializer. To give a value type a hand-authored JSON shape (readable import/export), derive from `ThemeValueJsonConverter<T>` and register it with `ThemeValueJson.Register`, usually from a method marked with `[RuntimeInitializeOnLoadMethod]`. A registered converter overrides the built-in converter and the fallback for its type, and — because the whole value graph is serialized through one serializer — it is applied **wherever `T` appears**: as the root value, nested inside another type, or inside a list, array, or dictionary.

```csharp
using derHugo.Themes;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using UnityEngine;

public sealed class ColorHexJsonConverter : ThemeValueJsonConverter<Color>
{
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.SubsystemRegistration)]
    private static void RegisterConverter()
    {
        ThemeValueJson.Register(new ColorHexJsonConverter());
    }

    protected override void WriteValue(JsonWriter writer, Color value, JsonSerializer serializer)
    {
        writer.WriteValue($"#{ColorUtility.ToHtmlStringRGBA(value)}");
    }

    protected override bool TryReadValue(JToken token, JsonSerializer serializer, out Color value, out string message)
    {
        value = default;
        message = string.Empty;

        if (token?.Type != JTokenType.String || !ColorUtility.TryParseHtmlString(token.Value<string>(), out value))
        {
            message = "Expected color HEX string.";
            return false;
        }

        return true;
    }
}
```

A registered converter is authoritative for its type: return `false` with a **message** from `TryReadValue` when the JSON is malformed (the message is surfaced to the import-review UI). You never handle nesting or object references yourself — write a nested themed value with `serializer.Serialize(writer, nested)` (it re-enters the same converter set) and write a `UnityEngine.Object` reference the same way (the engine encodes it into the object side-list, or an asset path for portable import/export). Value types **without** a converter fall back to Unity-like field serialization (public and `[SerializeField]` fields, matching `JsonUtility`'s rules), so most plain `[Serializable]` types need no converter at all. See the package converters under `Runtime/Json/BuiltIn Converters`.

The package ships built-in `ThemeValueJsonConverter<T>` implementations for common Unity value types under:

- `Assets/derHugo/Themes/Runtime/Json/BuiltIn Converters`

These give readable, stable JSON instead of Unity's default field names, and — like any converter — apply recursively when the type is nested inside another value:

- `ColorThemeValueJsonConverter`: exports a `#RRGGBBAA` HEX string.
- `Vector2/Vector3/Vector4ThemeValueJsonConverter` (and the `*Int` variants): export `{x, y, …}`.
- `QuaternionThemeValueJsonConverter`: exports its Euler angles `{x, y, z}`.
- `RectThemeValueJsonConverter`: exports `x`, `y`, `width`, and `height`.
- `BoundsThemeValueJsonConverter`: exports `center` and `extend`.
- `BoundsIntThemeValueJsonConverter`: exports `position` and `size`.
- `LayerMaskThemeValueJsonConverter`: exports only `bits`.
- `GradientThemeValueJsonConverter`: exports `mode`, `colorSpace`, `colorKeys`, and `alphaKeys`.
- `ColorBlockThemeValueJsonConverter`: exports its five HEX colors plus `colorMultiplier` and `fadeDuration`.

Use these files as copy/adapt references when you need a hand-authored JSON shape for a value type.

## Rename Safety

The theme value model is designed so renaming, moving, or removing a type keeps every reference working and preserves the stored data:

- **Definitions and stored values** are plain data keyed by GUID and by a rename-tolerant value-type identifier. If a value type's script is renamed or removed, the database keeps the stored value and the inspector shows a red type dropdown (the searchable picker) to re-point it; the value is preserved.
- **A `ThemeValue<T>` reference** stores its `DefinitionGuid` as plain data, so the theme link always survives. Its optional [modifier](#value-modifiers) is stored by a rename-tolerant identifier plus a JSON config; if the modifier's type is renamed or removed, the reference keeps its GUID and resolves the raw value, and the inspector shows a recovery dropdown of the remaining modifiers to re-select.
- **Converters** are explicit startup registrations and are not stored by the database, so renaming their class only requires updating the registration code.
- **Previews** are editor-only attribute registrations and are not stored by the database.

## Custom Binding Examples

Minimal custom binding:

```csharp
using UnityEngine;

namespace derHugo.Themes
{
    public sealed class LightEnabledBind : Bind<bool, Light>
    {
        protected override bool SetValue(bool value, Light target)
        {
            if (target.enabled == value)
            {
                return false;
            }

            target.enabled = value;
            return true;
        }

        protected override bool GetValue(Light target)
        {
            return target.enabled;
        }
    }
}
```

Transition-capable custom binding:

```csharp
using System.Collections;
using UnityEngine;
using UnityEngine.UI;

namespace derHugo.Themes
{
    public sealed class GraphicTintBind : Bind<Color, Graphic>
    {
        protected override bool SetValue(Color value, Graphic target)
        {
            if (target.color == value)
            {
                return false;
            }

            target.color = value;
            return true;
        }

        protected override Color GetValue(Graphic target)
        {
            return target.color;
        }

        protected override IEnumerator SetValueSmooth(Color targetValue, Graphic target, float fadeDuration)
        {
            var initial = target.color;
            if (fadeDuration <= 0f)
            {
                target.color = targetValue;
                yield break;
            }

            for (var t = 0f; t < fadeDuration; t += Time.deltaTime)
            {
                target.color = Color.Lerp(initial, targetValue, t / fadeDuration);
                yield return null;
            }

            target.color = targetValue;
        }
    }
}
```

## Optional Localization API

Namespace: `derHugo.Themes.Localization`

When `com.unity.localization` is installed, these become available:

- `BindLocalizedStringToTextMeshPro : Bind<LocalizedString, TMP_Text>`

More details: [Localization](Localization.md)

## JSON Serialization Limitations

JSON support is designed for database backup, transfer, and single-theme import/export. It is not a guaranteed serializer for arbitrary value types.

Conversion order (keyed by value type):

1. A registered [`ThemeValueJsonConverter<T>`](#custom-json-serialization) (custom or built-in) converts the value.
2. `UnityEngine.Object` values are serialized through the object side-list (asset paths in portable exports).
3. Unity-like field serialization (matching `JsonUtility`'s rules) runs for all remaining values.

Built-in converters cover:

- plain scalars (`float`, `int`, `bool`, `string`, …) as JSON primitives and enums as string names
- `Color` as HEX strings (`#RRGGBBAA`) and `ColorBlock` as named color-state HEX strings plus `colorMultiplier` and `fadeDuration`
- `Vector2`/`Vector3`/`Vector4` and `Vector2Int`/`Vector3Int` as `{x, y, …}`
- `Quaternion` as Euler-angle objects, `Rect`, `Bounds`, `BoundsInt`, `LayerMask`, and `Gradient`

See [Sample - JSON Converter References](Sample.md#json-converter-references) for the complete built-in converter list and their exact shapes.

Everything else uses Unity-like field serialization (public and `[SerializeField]` fields, matching `JsonUtility`'s rules). This keeps JSON support aligned with what Unity can serialize in the Inspector.

If that field serialization produces no data (`""` or `{}`), export/import reports that no theme JSON serialization is currently available for that type.

`UnityEngine.Object` root values are serialized as asset paths, with sub-assets as `path#ObjectName`.

Runtime single-theme import cannot resolve non-empty `UnityEngine.Object` references; object reference import is editor-only. For custom data that must round-trip through JSON, prefer small DTOs made of primitive fields or add a dedicated serialization path.

More details: [ThemeDatabase - JSON Serialization Limitations](ThemeDatabase.md#json-serialization-limitations)

## Choose the Right Extension Point

- Need to theme a new data kind: just use `ThemeValue<YourType>` - no subclass required.
- Need a hand-authored JSON shape for a value type: implement `ThemeValueJsonConverter<T>` and register it with `ThemeValueJson.Register`.
- Need local post-processing of a resolved value: implement `ThemeValueModifier<T>`.
- Need a compact inspector swatch for a value type: implement `ThemeValuePreview<T>` (`[ThemeValuePreview]`).
- Need to apply existing themed data to a custom target/component: implement `Bind<TValue, TTarget>` (`SetValue` + `GetValue`).
- Need animated transitions between values: override `Bind<TValue, TTarget>.SetValueSmooth`.

Related pages:

- [Getting Started](GettingStarted.md)
- [ThemeDatabase](ThemeDatabase.md)
- [Sample](Sample.md)

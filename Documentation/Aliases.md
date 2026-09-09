Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - Aliases</h1>

Aliases let a [`ThemeValue<T>`](ScriptingAPI.md#themevalue-and-value-types) reference a value by a stable, human-readable **key** instead of a definition GUID. A binding can then resolve against a consuming project's own [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) - even before the value exists.

## Contents

- [Why Aliases](#why-aliases)
- [Referencing a Value by Key](#referencing-a-value-by-key)
- [The Aliases Section](#the-aliases-section)
  - [Usage Scanning](#usage-scanning)
  - [Missing Keys](#missing-keys)
  - [Mapping and Unmapped Aliases](#mapping-and-unmapped-aliases)
  - [Automatic Rescan](#automatic-rescan)
- [Reference States](#reference-states)
- [Build Validation](#build-validation)
- [Runtime API](#runtime-api)
- [Related](#related)

---

## Why Aliases

A value [definition GUID](ThemeDatabase.md#data-model) is local to the database that created it. A binding pre-configured somewhere else - shipped inside a read-only package, an asset-store prefab, or a template scene - has no way to know a consuming project's GUIDs, so its theme link would break the moment it is dropped into a different project with a different [`ThemeDatabase`](ThemeDatabase.md)

An alias key is the portable layer that fixes this:

- The binding references a **name** (for example `Primary Color`), and the consuming project decides which of its own definitions that name maps to.
- An alias may be **mapped** (it points at a definition) or **unmapped** (it exists with an expected value type but no target yet), so a key can be referenced *before* the value is authored and reconciled later.
- The key is a stable contract other assets can depend on even without a target mapped value existing in the current [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) (yet).

## Referencing a Value by Key

Every [`ThemeValue<T>`](ScriptingAPI.md#themevalue-and-value-types) field uses the same searchable value picker - a [binding](Bind.md#selecting-a-value)'s state row, or a themed field on your own component. Besides the database definitions of the field's type, the picker offers the compatible **aliases**; selecting one stores its **key** (which takes precedence over a GUID).

- **Create an alias inline** - type a name in the picker's search and choose **`Create alias "<name>"…`**. It creates the alias on the active database and references it by key immediately; it stays unmapped until you map it in [the Aliases section](#the-aliases-section).
- **Add a value instead** - the picker's **`Add new value`** action still creates a concrete definition and selects it by GUID.

## The Aliases Section

Aliases are managed from the **Aliases** foldout of the [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) editor, available from every entry point ([asset inspector](ThemeDatabase.md#editor-entry-points), `Window > derHugo > Themes`, and `Edit > Project Settings > derHugo > Themes`), below **Constant Values** and **Themes**. The section header carries an **issues badge** (`N issues`) whenever there are unresolved entries.

[![][1]][1]

Each alias row lays out like a [value entry](ThemeDatabase.md#inspector-workflow): an optional **used-by** foldout, the **key**, the **expected type**, a remap arrow, the **target-value** mapping dropdown, and a delete (`X`).

### Usage Scanning

**Rescan Usages** discovers where keys are referenced across the assets a build would include:

- the enabled **Build Settings** scenes and, when the Addressables package is installed, the **Addressable** scenes
- every currently **open scene** and the current **prefab stage**
- and the recursive **prefab / ScriptableObject dependency closure** of all of the above.

The scan is **non-destructive** - closed scenes are opened and closed again without ever saving, so it leaves the project (and your open scenes) exactly as it found them. Expand a row's **used-by** handle to list the consumer sites; a site links to the live object when its scene or prefab is open, otherwise it pings the containing asset. The site label is condensed to the asset name, with the full path on its tooltip.

### Missing Keys

Keys that bindings reference but that have **no alias yet** are surfaced at the top of the list in red, with the discovered usage count. Resolve them in place:

[![2][]][2]

- **Register (`+`)** turns a missing key into a real alias (auto-mapped to the type's [default value](ThemeDatabase.md#favorite-default-value-per-type) when there is one), which you can then map or refine.
- **`Register N missing`** in the footer registers every discovered missing key at once.

### Mapping and Unmapped Aliases

An **unmapped** alias - one with no target, or whose target definition was deleted - shows a red `<None>`. Pick a value from its mapping dropdown to map it (or **`Add new value`** straight from the dropdown). The **Show Issues Only** toggle folds resolved entries away so only the missing keys and unmapped aliases remain.

### Automatic Rescan

**Auto Scan** keeps the list current automatically when scenes, prefabs, bindings, or the build scene list change, and after each recompile. To stay cheap it only runs while the Aliases list is actually on screen, and it pauses during play mode, compilation, and test runs. Turn it off on very large projects and use **Rescan Usages** on demand instead.

## Reference States

A keyed [`ThemeValue<T>`](ScriptingAPI.md#themevalue-and-value-types) field reflects how its key resolves against the active database:

[![3][]][3]

- **Resolved** - the alias exists and maps to a value of the required type.
- **Unmapped** - the alias exists but has no target yet; map it in the Aliases section.
- **Unknown** - no alias with that key exists; create it in the Aliases section (or from the field's picker).
- **Type mismatch** - the alias resolves to a value of a different type than the field requires; the field shows the details inline.

When a key does not resolve, the binding simply keeps its current value - the reference is never lost, and reconciling the alias later fixes every consumer at once.

## Build Validation

A pre-build check scans the build closure once for keyed references whose alias is **missing** or **unmapped** and logs each one to the Console. In an interactive build it then shows a **Build anyway / Cancel** dialog, so you can fix the issues first or continue deliberately. Unattended / batch builds (CI) log the issues and proceed without prompting.

The check only covers **alias** references (missing or unmapped keys); GUID-mode references and unassigned values are out of scope.

## Runtime API

Namespace: `derHugo.Themes`

- `ThemeValue<T>.Key` - the referenced alias key; when set it takes precedence over `DefinitionGuid`.
- `ThemeValue<T>.TryGetValue(out T value)` - resolves the key through the active database's alias map (then the active theme), or the GUID when no key is set.
- `ThemeDatabase.TryGetValueByKey<T>(string key, out T value)` - resolve a value directly by alias key. Returns `false` when the alias is missing, unmapped, or resolves to an incompatible type.

## Related

- [ThemeDatabase](ThemeDatabase.md) - the data model, inspector workflow, and JSON format.
- [Bind Components](Bind.md) - selecting a value (definition or alias) on a binding.
- [Scripting API](ScriptingAPI.md) - the runtime value-access API.
- [Getting Started](GettingStarted.md) - first setup and a quick tutorial.

[1]: images/Aliases.png
[2]: images/Aliases_Issues.png
[3]: images/Bind_Aliases.png
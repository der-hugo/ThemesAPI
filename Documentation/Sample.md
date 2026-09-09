Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - Sample</h1>

The **ThemesExample** sample is a branded, all-in-one themed application that demonstrates the full [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) workflow: bindables, bindings, modifiers, and runtime theme switching.

> [!NOTE]
> The brand names, marks and identities in this sample are fictional and exist for demonstration purposes only.

## Contents

- [Additional Dependencies](#additional-dependencies)
- [Import the Sample](#import-the-sample)
- [Open the Scene](#open-the-scene)
- [Inspect the Included ThemeDatabase](#inspect-the-included-themedatabase)
- [Play Mode Theme Switching](#play-mode-theme-switching)
- [Live Editing and JSON Backup](#live-editing-and-json-backup)

---

## Additional Dependencies

In addition to the minimal core package dependencies, the included sample is also using

- uGUI and TextMeshPro (`com.unity.ugui` includes both on Unity 6; older uGUI versions also need `com.unity.textmeshpro`). These are optional for the core, but required for this sample.
- Universal Render Pipeline (for the 3D mesh material and the themed scene light/camera background - alternatively change the `Product Mesh` material shader to e.g. `Standard` and adjust the scene bindings)
- Input System (for UI interactions - alternatively add the `StandaloneInputModule` to the `EventSystem`)

The exported Asset Store sample unitypackage declares these dependencies and requests installation when imported. For the UPM sample, install these packages in Package Manager before importing the sample.

The sample also bundles a set of open-source (SIL OFL 1.1) fonts used for the per-brand typography - see the package [Third-Party Notices](../Third-Party%20Notices.txt).

## Import the Sample

Find

    Assets/derHugo/Themes/Samples/ThemesExample.unitypackage

and double click it in order to import the sample

As the sample includes its own `ThemeDatabase` instance you have to activate it.

Find

    Assets/Samples/derHugo - Themes/<version>/ThemesExample/Runtime/Themes/SampleThemeDatabase.asset

[![][1]][1]

and in the Inspector click `Activate this Database`.

## Open the Scene

Open:

    Assets/Samples/derHugo - Themes/<version>/ThemesExample/Runtime/Scenes/ThemesShowcase.unity

[![][2]][2]

The scene is laid out like a real product screen:

- a **header** with the brand logo/name, a search field, a brand selector, a light/dark switch, and a "Spin" action
- a **left panel** with a mock hierarchy, the live theme color tokens, and a corner-radius demo
- a **center viewport** that is a genuine window into the live 3D scene (not an image) - a themed, orbiting mesh over themed lighting and background
- a **right panel** ("Inspector") showcasing every Unity built-in selectable - buttons (primary/secondary/ghost/danger), toggles (checkbox, switch, segmented), a TMP input field, a TMP dropdown, a slider, and status chips

**Note:** The sample uses `TextMeshPro` for the UI. If right after the sample import you see `TextMeshPro` related `NullReferenceExceptions` ensure the TextMeshPro Essentials are imported. You can trigger this by simply selecting one of the according Text objects to force an Inspector initialization for them.

The hierarchy includes static and interactive controls already wired with bindables and themed bindings, plus the 3D scene objects (mesh, lights, ground) that are themed through custom bindings.

## Inspect the Included ThemeDatabase

Sample [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) asset:

    Assets/Samples/derHugo - Themes/<version>/ThemesExample/Runtime/Themes/SampleThemeDatabase.asset

[![][1]][1]

It contains:

- value definitions (colors, floats, sprites, fonts and strings)
- constant values shared across all themes
- **6 themes** across **3 fictional brands** (`Auralis`, `Emberline`, `Papyra`), each in a **light** and a **dark** variant
- the current active theme (`Auralis Dark`)

You can edit it via:

- asset inspector
- `Window > derHugo > Themes`
- `Edit > Project Settings > derHugo > Themes`

## Play Mode Theme Switching

Enter Play Mode and switch themes using the sample UI controls: pick a brand in the header's segmented control and flip the light/dark switch.

[![][5]][5]

The whole application - UI, typography, corner radius and the 3D viewport - re-themes live.

[![][3]][3]

## Live Editing and JSON Backup

You can edit theme values in edit mode or play mode and immediately see updates on bound UI.

[![][4]][4]

The sample includes JSON backup examples:

- `Assets/Samples/derHugo - Themes/<version>/ThemesExample/SampleThemeDatabase.json`: complete database snapshot.
- `Assets/Samples/derHugo - Themes/<version>/ThemesExample/Auralis Dark.json`: single theme snapshot.

Use the database tooling to:

- export the current database to JSON
- export or import one theme through the **Themes** list controls
- import a complete database into the current database
- import a complete database as a new asset for review

Related pages:

- [Getting Started](GettingStarted.md)
- [ThemeDatabase](ThemeDatabase.md)
- [Scripting API](ScriptingAPI.md)

[1]: images/sample/ThemeDatabase.png
[2]: images/sample/Scene.png
[3]: images/sample/Themed_UI.png
[4]: images/sample/LiveEdit.gif
[5]: images/sample/ThemeSwitches.png

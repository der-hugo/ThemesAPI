Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - Sample</h1>

The sample demonstrates the full [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) workflow: bindables, bindings, and runtime theme switching.

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

- Universal Render Pipeline (For the 3D cube material - alternatively simply change the Shader of the `Cube` material to e.g. `Standard`)
- Input System (For UI interactions - alternatively add the `StandaloneInputModule` to the `EventSystem`)

## Import the Sample

In Package Manager:

1. Select `derHugo - Themes`
2. Import sample `Themes Example`

As the sample includes its own `ThemeDatabase` instance you have to activate it.

Find

    Assets/Samples/derHugo - Themes/<version>/Themes Example/Runtime/Themes/SampleThemeDatabase.asset

[![][3]][3]

and in the Inspector click `Activate this Database`.

## Open the Scene

Open:

    Assets/Samples/derHugo - Themes/<version>/Themes Example/Runtime/Scenes/ThemesExample.unity

[![][1]][1]

**Note:** The sample uses `TextMeshPro` for the UI. If right after the sample import you see `TextMeshPro` related `NullReferenceExceptions` ensure the TextMeshPro Essentials are imported. You can trigger this by simply selecting one of the according Text objects to force an Inspector initialization for them.

The hierarchy includes static and interactive controls already wired with bindables and themed bindings.

[![][2]][2]

## Inspect the Included ThemeDatabase

Sample [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) asset:

    Assets/Samples/derHugo - Themes/<version>/Themes Example/Runtime/Themes/SampleThemeDatabase.asset

[![][3]][3]

It contains:

- value definitions
- constant values
- two themes (`Light`, `Dark`)
- an active theme

You can edit it via:

- asset inspector
- `Window > derHugo > Themes`
- `Edit > Project Settings > derHugo > Themes`

## Play Mode Theme Switching

Enter Play Mode and switch themes using the sample UI controls (including [`ThemeDropdown`](ScriptingAPI.md#theme-switching-helpers) and apply actions).

[![][4]][4]
[![][5]][5]

## Live Editing and JSON Backup

You can edit theme values in edit mode or play mode and immediately see updates on bound UI.

[![][6]][6]

The sample includes JSON backup examples:

- `Assets/Samples/derHugo - Themes/<version>/Themes Example/ThemeDatabaseBackup.json`: complete database snapshot.
- `Assets/Samples/derHugo - Themes/<version>/Themes Example/Example Dark Theme - Backup.json`: single theme snapshot.

Use the database tooling to:

- export the current database to JSON
- export or import one theme through the **Themes** list controls
- import a complete database into the current database
- import a complete database as a new asset for review

Related pages:

- [Getting Started](GettingStarted.md)
- [ThemeDatabase](ThemeDatabase.md)
- [Scripting API](ScriptingAPI.md)

[1]: images/sample/Scene.png
[2]: images/sample/Hierarchy.png
[3]: images/sample/ThemeDatabase.png
[4]: images/sample/Sample_Light.png
[5]: images/sample/Sample_Dark.png
[6]: images/sample/live_edit.gif

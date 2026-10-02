Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - Project Settings</h1>

`Edit > Project Settings > derHugo > Themes` provides project-wide access to the currently active [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) and its tooling.

## Contents

- [Open the Settings Page](#open-the-settings-page)
- [Project-Wide Active Database](#project-wide-active-database)
- [Active Database Resolution Behavior](#active-database-resolution-behavior)
- [JSON Import and Export](#json-import-and-export)
- [Usage scan settings](#usage-scan-settings-header)
- [Related Windows and Inspectors](#related-windows-and-inspectors)

---

## Open the Settings Page

Open:

- `Edit > Project Settings > derHugo > Themes`

This page embeds the same database inspector tooling used by the asset inspector and the dedicated window.

## Project-Wide Active Database

At the top of the page, **Project-wide Database** lets you view or switch the active database asset.

Important behavior:

- Themes works with a single central active [ThemeDatabase](ThemeDatabase.md). Clearing the field disables theming without changing the latest applied binding values or references.
- Switching to a different database can invalidate existing definition GUID references; the editor asks for confirmation.
- It is written into `PlayerSettings` preloaded assets **during a build** so a built player can resolve it on runtime.

More details: [ThemeDatabase - Active Database Lifecycle](ThemeDatabase.md#active-database-lifecycle)

## Active Database Resolution Behavior

### When no database is assigned

The field stays empty. Click **Create New...** to choose where to save a new database and activate it. Canceling does nothing. You can also browse existing assets directly through the object field. The package never creates a database automatically or forces a selection popup.

Consumers show a missing-database error and retain the shortcut to the Themes window. Bindings keep their current values and references until a database is assigned.

### When deleting the active database

The active selection is cleared. No replacement is selected automatically.

## JSON Import and Export

You can run JSON actions directly from this settings page and the shared database inspector:

- Top-level `Export JSON`: exports the complete active database snapshot.
- Top-level `Import JSON`: imports a complete database snapshot.
- **Themes** list `+` dropdown > `Import Theme`: imports a single theme JSON file.
- Theme header `Export JSON`: exports only that theme.

Full database import modes:

- **Import Into This Database**: replaces current content.
- **Import As New Database Asset**: creates a new asset for review.

Theme import modes:

- **Import And Activate**: adds the imported theme and makes it active.
- **Import Only**: adds the imported theme without switching the active theme.

More details: [ThemeDatabase - JSON Format and Import/Export](ThemeDatabase.md#json-format-and-importexport)

## Usage scan settings (header)

Directly under the database and JSON actions, the header hosts the shared usage-scan controls (kept here so the lists stay uncluttered):

- **Rescan** (↻) - runs the usage scan once now (feeds both the Aliases list and the Issues list).
- **Auto Scan** - project setting (`AutoScanUsages`, on by default): rebuild automatically as scenes, prefabs, bindings, or the build scene list change and after each recompile, while a list is on screen.
- **Validate theme usages before build** (on by default) - the pre-build check below.

### Validate Theme Usages Before Build

The pre-build check scans every build-included asset - scenes, prefabs, ScriptableObjects, and preloaded assets - for [`ThemeValue`](ScriptingAPI.md) references that will not resolve against the active database:

- **Missing / unmapped aliases** - a keyed reference whose alias does not exist, or exists but maps to nothing.
- **Alias / value type conflicts** - a keyed or GUID reference that resolves to a definition of a type the field cannot accept.
- **Missing value references** - a GUID reference whose definition no longer exists.
- **Unassigned values** - a field left at "None" that is actually read at runtime. This is context-aware for bindings: a non-`Normal` selectable state inherits `Normal`, so only a used block's `Normal` (or a standalone / third-party field) is flagged.

Unresolved usages are always logged to the Console as **errors**. In an interactive build the Themes settings page opens (so you can inspect the Issues list, see below) and a dialog lets you cancel the build or build anyway; an unattended (batch/CI) build only logs and proceeds without failing (a logged error does not fail a headless build). Disable the toggle - temporarily or permanently - for projects whose value types need handling the check would otherwise flag.

### Issues list

The database inspector (this page, the Themes window, and the asset inspector) shows an **Issues** list at the **top**, fed by the **same scan** as the Aliases list and classified against the active database at draw time (so it updates immediately when you fix an alias or definition - no rescan needed). It is **hidden entirely when there is nothing to show**, and carries a count badge like the Aliases list.

- **Errors** (red) come first: one entry per affected component or ScriptableObject asset, reduced to the GameObject/asset name (with the component type), combining all of that host's issues into one row. Clicking a row **pings the live instance** when its scene or prefab stage is loaded, otherwise the **containing asset**.
- **Warnings** (yellow) follow: an **unmapped alias that no binding uses** is harmless for a build (nothing resolves against it), so it is revealed as a warning rather than an error - in both the Issues list and the Aliases badge - and it never blocks a build.

## Related Windows and Inspectors

- `Window > derHugo > Themes`
- [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) asset inspector

All of these surfaces share the same core editing workflow.

Related pages:

- [Getting Started](GettingStarted.md)
- [ThemeDatabase](ThemeDatabase.md)
- [Scripting API](ScriptingAPI.md)

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

- **Refresh icon** (↻) - runs the shared usage scan once now, updating [value usage counts](ThemeDatabase.md#value-usages), [alias usage counts](Aliases.md#usage-view-toggle), and Validation. The Aliases header has the same refresh action.
- **Settings icon** - opens the shared options: **Auto Scan Usages** (`AutoScanUsages`, on by default) and **Validate Usages Before Build** (on by default).
- **Auto Scan Usages** rebuilds automatically as scenes, prefabs, bindings, or the build scene list change and after each recompile, while the database editor or its scan results are displayed. It pauses during play mode, compilation, and test runs.

Each value and alias row uses the same compact **search-icon usage toggle** to show consumer links. Counts above 99 display as `99+`; hover for the exact count. With no discovered usages, the icon-only button is disabled and its list is hidden. The row buttons do not run individual scans; use the header refresh action when the results are stale or no scan has completed.

### Validate Theme Usages Before Build

The pre-build check uses the [shared scan scope](Aliases.md#usage-scanning) to find [`ThemeValue`](ScriptingAPI.md#themevalue-and-value-types) and [`ThemedFloat`](ScriptingAPI.md#themedfloat) references that will not resolve against the active database, including references inside selected modifier configurations:

- **Missing / unmapped aliases** - a keyed reference whose alias does not exist, or exists but maps to nothing.
- **Alias / value type conflicts** - a keyed or GUID reference that resolves to a definition of a type the field cannot accept.
- **Missing value references** - a GUID reference whose definition no longer exists.
- **Unassigned values** - a reference left at "None" where an assignment is required. For `ThemeValue` selectable states, non-`Normal` states can inherit `Normal`; `Values.Normal` is required in every binding mode, and `OffValues.Normal` is required in Toggle mode. Standalone references are also checked.
- **Unassigned themed floats** - `ThemedFloat` in **Themed** mode requires a float reference, including fade durations and modifier factors. An empty selection is reported even when runtime code has a fallback. **Custom** mode is ignored, including any stale stored GUID.

Unresolved usages are always logged to the Console as **errors**. In an interactive build the Themes settings page opens (so you can inspect the Issues list, see below) and a dialog lets you cancel the build or build anyway; an unattended (batch/CI) build only logs and proceeds without failing (a logged error does not fail a headless build). Disable the toggle - temporarily or permanently - for projects whose value types need handling the check would otherwise flag.

### Issues list

The **Validation** header at the top of the active database editor (this page, the Themes window, and the asset inspector) shows issue/warning badges or an **OK / Not scanned** status. Expand it when issues are present to view the **Issues** list. The list shares the same scan as value and alias usages, and references are classified against the active database at draw time, so fixing an alias or definition does not require another asset scan. When there are no issues, the list is hidden and the status header remains visible.

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

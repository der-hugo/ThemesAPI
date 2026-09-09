Package entry point: [README](../README.md)

---

<h1><img src="images/colors.png" alt="Logo" width="80" align="middle" />&nbsp; Themes - Project Settings</h1>

`Edit > Project Settings > derHugo > Themes` provides project-wide access to the currently active [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) and its tooling.

## Contents

- [Open the Settings Page](#open-the-settings-page)
- [Project-Wide Active Database](#project-wide-active-database)
- [Active Database Resolution Behavior](#active-database-resolution-behavior)
- [JSON Import and Export](#json-import-and-export)
- [Related Windows and Inspectors](#related-windows-and-inspectors)

---

## Open the Settings Page

Open:

- `Edit > Project Settings > derHugo > Themes`

This page embeds the same database inspector tooling used by the asset inspector and the dedicated window.

## Project-Wide Active Database

At the top of the page, **Project-wide Database** lets you view or switch the active database asset.

Important behavior:

- Exactly one database is active at a time.
- Switching to a different database can invalidate existing definition GUID references; the editor asks for confirmation.
- The active database is synced to `PlayerSettings` preloaded assets so runtime can resolve it predictably.

More details: [ThemeDatabase - Active Database Lifecycle](ThemeDatabase.md#active-database-lifecycle)

## Active Database Resolution Behavior

### When no database exists

The editor creates a default database:

- `Assets/Resources/ThemeDatabase.asset`

### When multiple databases exist

If no single clear active candidate can be resolved, a selection popup asks you to choose one.

### When deleting the active database

On deletion, the package tries to resolve and activate another available database.

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

## Related Windows and Inspectors

- `Window > derHugo > Themes`
- [`ThemeDatabase`](ScriptingAPI.md#themedatabase-runtime-api) asset inspector

All of these surfaces share the same core editing workflow.

Related pages:

- [Getting Started](GettingStarted.md)
- [ThemeDatabase](ThemeDatabase.md)
- [Scripting API](ScriptingAPI.md)
